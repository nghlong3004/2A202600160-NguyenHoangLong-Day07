# Báo Cáo Lab 7: Embedding & Vector Store

**Họ tên:** Nguyễn Hoàng Long
**Nhóm:** C401-E6
**Ngày:** 2026-04-10

---

## 1. Warm-up (5 điểm)

### Cosine Similarity (Ex 1.1)

**High cosine similarity nghĩa là gì?**
> Cosine similarity đo góc giữa hai vector trong không gian embedding. Khi hai đoạn văn bản có cosine similarity cao (gần 1.0), nghĩa là chúng nằm gần nhau về mặt hướng trong không gian vector — tức là embedding model đánh giá chúng có ngữ nghĩa tương tự nhau.

**Ví dụ HIGH similarity:**
- Sentence A: "Python is a popular programming language for web development."
- Sentence B: "Python is widely used for building web applications."
- Tại sao tương đồng: Cả hai câu đều nói về Python trong bối cảnh phát triển web, dùng các từ khoá gần nghĩa ("programming language" ~ "building applications", "web development" ~ "web applications"). Embedding model sẽ ánh xạ chúng vào vùng gần nhau trong không gian vector.

**Ví dụ LOW similarity:**
- Sentence A: "The weather is sunny today."
- Sentence B: "Vector databases store embeddings for similarity search."
- Tại sao khác: Hai câu thuộc hai domain hoàn toàn khác biệt (thời tiết vs. công nghệ), không chia sẻ từ khoá hay ngữ nghĩa nào. Embedding vectors sẽ chỉ theo các hướng khác nhau trong không gian, cho cosine similarity gần 0.

**Tại sao cosine similarity được ưu tiên hơn Euclidean distance cho text embeddings?**
> Cosine similarity chỉ đo hướng (angle) của vector, không phụ thuộc vào độ dài (magnitude). Điều này quan trọng vì hai đoạn text có cùng ý nghĩa nhưng khác độ dài sẽ tạo ra embeddings có magnitude khác nhau — Euclidean distance sẽ đánh giá chúng xa nhau, trong khi cosine similarity vẫn cho điểm cao vì hướng vector giống nhau.

### Chunking Math (Ex 1.2)

**Document 10,000 ký tự, chunk_size=500, overlap=50. Bao nhiêu chunks?**
> Áp dụng công thức: `num_chunks = ceil((doc_length - overlap) / (chunk_size - overlap))`
> `num_chunks = ceil((10000 - 50) / (500 - 50)) = ceil(9950 / 450) = ceil(22.11) = 23 chunks`

**Nếu overlap tăng lên 100, chunk count thay đổi thế nào? Tại sao muốn overlap nhiều hơn?**
> `num_chunks = ceil((10000 - 100) / (500 - 100)) = ceil(9900 / 400) = ceil(24.75) = 25 chunks`
> Overlap tăng từ 50 lên 100 → chunk count tăng từ 23 lên 25. Overlap lớn hơn giúp giảm nguy cơ mất ngữ cảnh tại ranh giới giữa hai chunk — một câu hoặc ý quan trọng nằm ở rìa chunk sẽ được lặp lại trong chunk tiếp theo, giúp retrieval tìm được thông tin đầy đủ hơn. Tuy nhiên, đổi lại là chi phí lưu trữ và tính toán tăng lên.

---

## 2. Document Selection — Nhóm (10 điểm)

### Domain & Lý Do Chọn

**Domain:** Sinh học & Y sinh (Biology & Biomedical Sciences)

**Tại sao nhóm chọn domain này?**
> Nhóm chọn domain Sinh học vì: (1) tài liệu phong phú, dễ tiếp cận dưới dạng fact sheets, white papers, và lecture notes từ các tổ chức y tế uy tín, (2) nội dung đa dạng từ di truyền học cơ bản (Mendel) đến ứng dụng lâm sàng (NIPT, cancer screening), tạo điều kiện tốt cho việc test metadata filtering theo topic, và (3) các tài liệu Markdown có cấu trúc rõ ràng (sections, headings, bullet points) phù hợp để so sánh chunking strategies.

### Data Inventory

| # | Tên tài liệu | Nguồn | Số ký tự | Metadata đã gán |
|---|-------------|-------|----------|-----------------|
| 1 | PrenatalGenome WhitePaper | prenatalgenome.it | ~13,800 | `topic: prenatal_testing, lang: en, type: whitepaper` |
| 2 | Alpha Thalassemia Fact Sheet | squarespace.com (genetics org) | ~7,500 | `topic: genetic_disorder, lang: en, type: factsheet` |
| 3 | Mendelian Inheritance Lecture | uomus.edu.iq | ~5,700 | `topic: genetics, lang: en, type: lecture` |
| 4 | Brain Tumours in Children | cclg.org.uk | ~11,300 | `topic: oncology, lang: en, type: factsheet` |
| 5 | Introduction to Molecular Diagnostics | epemed.org (AdvaMedDx) | ~11,400 | `topic: diagnostics, lang: en, type: guide` |
| 6 | Non-invasive Prenatal Testing (NIPT) | genetics.edu.au | ~4,700 | `topic: prenatal_testing, lang: en, type: factsheet` |
| 7 | Cancer Screening Action Guide | nachc.org (CDC) | ~6,100 | `topic: oncology, lang: en, type: guide` |

### Metadata Schema

| Trường metadata | Kiểu | Ví dụ giá trị | Tại sao hữu ích cho retrieval? |
|----------------|------|---------------|-------------------------------|
| `topic` | string | `prenatal_testing`, `genetic_disorder`, `genetics`, `oncology`, `diagnostics` | Cho phép filter theo chủ đề y sinh cụ thể — khi câu hỏi về ung thư, chỉ search trong docs `oncology`, giảm nhiễu từ tài liệu di truyền/xét nghiệm |
| `type` | string | `factsheet`, `whitepaper`, `lecture`, `guide` | Phân biệt loại tài liệu — factsheet thường ngắn gọn và trả lời câu hỏi trực tiếp, guide thì chi tiết hơn. Giúp chọn đúng độ sâu thông tin cho từng query |
| `lang` | string | `en` | Dự phòng cho mở rộng đa ngôn ngữ — hiện tại tất cả tài liệu đều tiếng Anh nhưng có thể thêm tài liệu tiếng Việt sau |

---

## 3. Chunking Strategy — Cá nhân chọn, nhóm so sánh (15 điểm)

### Baseline Analysis

Chạy `ChunkingStrategyComparator().compare()` trên 3 tài liệu Markdown sinh học (chunk_size=500, overlap=50):

| Tài liệu | Strategy | Chunk Count | Avg Length | Preserves Context? |
|-----------|----------|-------------|------------|-------------------|
| PrenatalGenome WhitePaper (~13,800 chars) | FixedSizeChunker (`fixed_size`) | 31 | 493.6 | Trung bình — cắt giữa câu y khoa dài (kết quả PCR, protocol) |
| PrenatalGenome WhitePaper | SentenceChunker (`by_sentences`) | 47 | 291.8 | Giữ câu trọn vẹn, nhưng tạo nhiều chunks nhỏ do whitepaper có nhiều câu ngắn xen kẽ |
| PrenatalGenome WhitePaper | RecursiveChunker (`recursive`) | 35 | 392.5 | Tốt — tách theo paragraph, giữ trọn mô tả quy trình xét nghiệm |
| Alpha Thalassemia (~7,500 chars) | FixedSizeChunker (`fixed_size`) | 17 | 488.5 | Trung bình — cắt giữa mô tả triệu chứng và phân loại bệnh |
| Alpha Thalassemia | SentenceChunker (`by_sentences`) | 18 | 414.4 | Tốt — giữ câu trọn vẹn, chunks đều đặn hơn so với whitepaper |
| Alpha Thalassemia | RecursiveChunker (`recursive`) | 19 | 393.1 | Tốt — tách theo sections (Symptoms, Treatment, Diagnosis) |
| Mendelian Inheritance (~5,700 chars) | FixedSizeChunker (`fixed_size`) | 13 | 486.0 | Trung bình — cắt giữa định nghĩa thuật ngữ |
| Mendelian Inheritance | SentenceChunker (`by_sentences`) | 21 | 270.0 | Nhiều chunks nhỏ — lecture có câu ngắn, tạo fragments rời rạc |
| Mendelian Inheritance | RecursiveChunker (`recursive`) | 14 | 406.6 | Tốt — tách theo phần bài giảng (Laws, Terms, Patterns) |

### Strategy Của Tôi

**Loại:** RecursiveChunker (với chunk_size=500)

**Mô tả cách hoạt động:**
> RecursiveChunker hoạt động bằng cách thử tách text theo thứ tự ưu tiên separator: `\n\n` (paragraph) → `\n` (line break) → `. ` (sentence) → ` ` (word) → `""` (character). Với mỗi separator, nó split text, gom các phần nhỏ lại nếu tổng vẫn dưới chunk_size, và đệ quy vào phần quá lớn với separator tiếp theo. chunk_size=500 được chọn vì tài liệu sinh học thường có câu dài, cần chunk lớn hơn để giữ trọn vẹn ý.

**Tại sao tôi chọn strategy này cho domain nhóm?**
> Domain sinh học có tài liệu Markdown với cấu trúc rõ ràng: sections ("What is...", "Symptoms", "Treatment"), paragraphs mô tả bệnh lý, và bullet points liệt kê triệu chứng. RecursiveChunker tận dụng cấu trúc này bằng cách ưu tiên tách theo paragraph (`\n\n`) — mỗi paragraph trong factsheet thường chứa 1 khái niệm y khoa hoàn chỉnh. Dữ liệu thực tế cho thấy SentenceChunker tạo quá nhiều chunks nhỏ trên whitepaper (47 chunks, avg 291.8) và lecture (21 chunks, avg 270.0), trong khi RecursiveChunker giữ chunks ở mức 35 và 14 chunks với avg length ~400 chars — phù hợp hơn cho retrieval.

### So Sánh: Strategy của tôi vs Baseline

| Tài liệu | Strategy | Chunk Count | Avg Length | Retrieval Quality? |
|-----------|----------|-------------|------------|-------------------|
| PrenatalGenome WhitePaper | best baseline (fixed_size) | 31 | 493.6 | Chunks đều nhưng cắt giữa giải thích kỹ thuật cfDNA extraction, sequencing protocol |
| PrenatalGenome WhitePaper | **RecursiveChunker (của tôi)** | 35 | 392.5 | Tách tốt theo paragraphs — mỗi chunk chứa 1 bước trong quy trình niFES hoàn chỉnh |
| Alpha Thalassemia | best baseline (by_sentences) | 18 | 414.4 | Chunks tự nhiên, giữ câu trọn vẹn, phù hợp factsheet |
| Alpha Thalassemia | **RecursiveChunker (của tôi)** | 19 | 393.1 | Gom nhiều câu liên quan vào 1 chunk, giữ ngữ cảnh section (Symptoms gồm cả HbH + Major) |

### So Sánh Với Thành Viên Khác

| Thành viên | Strategy | Retrieval Score (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| Nguyễn Hoàng Long | RecursiveChunker (chunk_size=500) | 7 | Giữ ngữ cảnh theo paragraph, phù hợp tài liệu y khoa có cấu trúc | Chunks có thể quá lớn cho factsheet câu ngắn |
| Khổng Mạnh Tuấn | fixed_size | 8 | Ổn định, dễ kiểm soát chunk | Query 4/5 vẫn lệch tài liệu kỳ vọng, chỉ trả lời được câu hỏi dễ, các câu hỏi phức tạp chọn đúng tài liệu và chỉ số liên quan cao  nhưng câu trả lời không chính xác ( model local ) |
| Lâm Hoàng Hải | sentence | 7 | Dễ cài đặt | Chỉ đúng 4 / 5 query, miss ý của câu hỏi, trả lời đúng câu hỏi đơn giản (chỉ cần 1 đáp án), nhưng fail câu hỏi phức tạp, cần suy luận từ tương đồng, ngữ cảnh, tốn nhiều chunk hơn phương pháp khác |


**Strategy nào tốt nhất cho domain này? Tại sao?**
> RecursiveChunker là strategy phù hợp nhất cho domain Sinh học & Y sinh vì: (1) tài liệu có cấu trúc rõ ràng với sections, headings, paragraphs — RecursiveChunker tận dụng các separator tự nhiên (`\n\n`, `\n`) để tách theo đúng ranh giới nội dung; (2) kết hợp tốt cả factsheet (câu ngắn, gom lại thành chunks có ý nghĩa) và whitepaper (paragraph dài, tách theo đúng bước quy trình); (3) SentenceChunker tốt cho factsheet nhưng kém cho whitepaper có câu dài; FixedSizeChunker ổn định nhưng phá vỡ ngữ cảnh y khoa quan trọng.

---

## 4. My Approach — Cá nhân (10 điểm)

Giải thích cách tiếp cận của bạn khi implement các phần chính trong package `src`.

### Chunking Functions

**`SentenceChunker.chunk`** — approach:
> Sử dụng regex `re.split(r'(?<=[.!?])[\s]', text)` để phát hiện ranh giới câu — split sau các dấu `.`, `!`, `?` khi theo sau là bất kỳ whitespace nào (space, newline, tab). Sau đó strip whitespace thừa và lọc câu rỗng. Gom các câu thành groups theo `max_sentences_per_chunk` bằng cách lặp qua danh sách câu với bước nhảy = max_sentences, join bằng space. Edge case: text rỗng trả `[]`, câu cuối cùng được giữ nguyên dù group chưa đầy.

**`RecursiveChunker.chunk` / `_split`** — approach:
> Algorithm hoạt động theo pattern đệ quy: `_split(text, separators)` kiểm tra base case (text <= chunk_size => return ngay). Nếu không, lấy separator đầu tiên, split text, rồi merge các phần nhỏ lại. Phần nào vẫn quá lớn sẽ đệ quy với danh sách separator còn lại. Base case cuối cùng: hết separator thì force-split theo chunk_size. Edge case đặc biệt: separator rỗng `""` được xử lý như force-split.

### EmbeddingStore

**`add_documents` + `search`** — approach:
> `_make_record()` tạo dict chuẩn hoá gồm `id`, `content`, `embedding` (gọi embedding_fn), và `metadata` (merge metadata gốc + doc_id). `add_documents` lặp qua từng doc, tạo record rồi append vào `self._store` (và thêm vào ChromaDB nếu có). `search` gọi `_search_records` — embed query, tính dot product với mọi stored embedding, sort giảm dần, trả top_k.

**`search_with_filter` + `delete_document`** — approach:
> `search_with_filter` filter trước rồi search sau: lọc `self._store` bằng dict comprehension kiểm tra mọi key-value trong `metadata_filter`, rồi gọi `_search_records` trên danh sách đã filter. `delete_document` dùng list comprehension giữ lại các record có `doc_id != target`, so sánh size trước/sau để return True/False.

### KnowledgeBaseAgent

**`answer`** — approach:
> `__init__` lưu reference đến `store` và `llm_fn`. `answer()` thực hiện 3 bước RAG: (1) gọi `store.search(question, top_k)` lấy chunks, (2) build prompt dạng `"Context:\n[1] chunk1\n[2] chunk2\n...\nQuestion: ...\nAnswer:"`, (3) gọi `llm_fn(prompt)` và return kết quả. Prompt được thiết kế để LLM trả lời dựa trên context, giảm hallucination.

### Test Results

```
tests/test_solution.py::TestProjectStructure::test_root_main_entrypoint_exists PASSED
tests/test_solution.py::TestProjectStructure::test_src_package_exists PASSED
tests/test_solution.py::TestClassBasedInterfaces::test_chunker_classes_exist PASSED
tests/test_solution.py::TestClassBasedInterfaces::test_mock_embedder_exists PASSED
tests/test_solution.py::TestFixedSizeChunker::test_chunks_respect_size PASSED
tests/test_solution.py::TestFixedSizeChunker::test_correct_number_of_chunks_no_overlap PASSED
tests/test_solution.py::TestFixedSizeChunker::test_empty_text_returns_empty_list PASSED
tests/test_solution.py::TestFixedSizeChunker::test_no_overlap_no_shared_content PASSED
tests/test_solution.py::TestFixedSizeChunker::test_overlap_creates_shared_content PASSED
tests/test_solution.py::TestFixedSizeChunker::test_returns_list PASSED
tests/test_solution.py::TestFixedSizeChunker::test_single_chunk_if_text_shorter PASSED
tests/test_solution.py::TestSentenceChunker::test_chunks_are_strings PASSED
tests/test_solution.py::TestSentenceChunker::test_respects_max_sentences PASSED
tests/test_solution.py::TestSentenceChunker::test_returns_list PASSED
tests/test_solution.py::TestSentenceChunker::test_single_sentence_max_gives_many_chunks PASSED
tests/test_solution.py::TestRecursiveChunker::test_chunks_within_size_when_possible PASSED
tests/test_solution.py::TestRecursiveChunker::test_empty_separators_falls_back_gracefully PASSED
tests/test_solution.py::TestRecursiveChunker::test_handles_double_newline_separator PASSED
tests/test_solution.py::TestRecursiveChunker::test_returns_list PASSED
tests/test_solution.py::TestEmbeddingStore::test_add_documents_increases_size PASSED
tests/test_solution.py::TestEmbeddingStore::test_add_more_increases_further PASSED
tests/test_solution.py::TestEmbeddingStore::test_initial_size_is_zero PASSED
tests/test_solution.py::TestEmbeddingStore::test_search_results_have_content_key PASSED
tests/test_solution.py::TestEmbeddingStore::test_search_results_have_score_key PASSED
tests/test_solution.py::TestEmbeddingStore::test_search_results_sorted_by_score_descending PASSED
tests/test_solution.py::TestEmbeddingStore::test_search_returns_at_most_top_k PASSED
tests/test_solution.py::TestEmbeddingStore::test_search_returns_list PASSED
tests/test_solution.py::TestKnowledgeBaseAgent::test_answer_non_empty PASSED
tests/test_solution.py::TestKnowledgeBaseAgent::test_answer_returns_string PASSED
tests/test_solution.py::TestComputeSimilarity::test_identical_vectors_return_1 PASSED
tests/test_solution.py::TestComputeSimilarity::test_opposite_vectors_return_minus_1 PASSED
tests/test_solution.py::TestComputeSimilarity::test_orthogonal_vectors_return_0 PASSED
tests/test_solution.py::TestComputeSimilarity::test_zero_vector_returns_0 PASSED
tests/test_solution.py::TestCompareChunkingStrategies::test_counts_are_positive PASSED
tests/test_solution.py::TestCompareChunkingStrategies::test_each_strategy_has_count_and_avg_length PASSED
tests/test_solution.py::TestCompareChunkingStrategies::test_returns_three_strategies PASSED
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::test_filter_by_department PASSED
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::test_no_filter_returns_all_candidates PASSED
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::test_returns_at_most_top_k PASSED
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::test_delete_reduces_collection_size PASSED
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::test_delete_returns_false_for_nonexistent_doc PASSED
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::test_delete_returns_true_for_existing_doc PASSED

============================= 42 passed in 0.38s ==============================
```

**Số tests pass:** 42 / 42

---

## 5. Similarity Predictions — Cá nhân (5 điểm)

| Pair | Sentence A | Sentence B | Dự đoán | Actual Score | Đúng? |
|------|-----------|-----------|---------|--------------|-------|
| 1 | Python is a programming language | Python is used for software development | high | 0.1250 | Sai — mock embedder không capture ngữ nghĩa |
| 2 | Machine learning trains models on data | Deep learning uses neural networks | high | -0.0518 | Sai — mock cho kết quả âm dù topic liên quan |
| 3 | The weather is sunny today | Vector databases store embeddings | low | 0.0141 | Đúng — gần 0 như dự đoán |
| 4 | Retrieval augmented generation uses context | RAG systems find relevant documents | high | -0.0341 | Sai — mock không hiểu "RAG" = "Retrieval augmented generation" |
| 5 | I love eating pizza | The cat sat on the mat | low | 0.1279 | Sai — mock cho score cao hơn pair 1 dù không liên quan |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn nghĩa?**
> Kết quả bất ngờ nhất là Pair 5 ("pizza" vs "cat") có similarity (0.1279) gần bằng Pair 1 ("Python programming" vs "Python development" — 0.1250), trong khi Pair 4 (RAG vs RAG rewritten — -0.0341) lại là âm. Điều này cho thấy mock embedder (dựa trên MD5 hash) hoàn toàn không capture ngữ nghĩa — nó chỉ tạo vector xác định theo chuỗi ký tự. Với real embedding model (ví dụ `all-MiniLM-L6-v2`), ta sẽ thấy Pair 1 và 4 có điểm cao hơn nhiều vì model được train để hiểu semantic similarity.

---

## 6. Results — Cá nhân (10 điểm)

Chạy 5 benchmark queries của nhóm trên implementation cá nhân của bạn trong package `src`. **5 queries phải trùng với các thành viên cùng nhóm.**

### Benchmark Queries & Gold Answers (nhóm thống nhất)

| # | Query | Gold Answer |
|---|-------|-------------|
| 1 | In NIPT, what is the role of paternal DNA information? | Paternal-inherited SNPs in cfDNA được phân tích để ước lượng fetal fraction và xác nhận phát hiện fetal DNA. Từ `01_Prenatal_Genome_White_Paper` (dòng 69-77) |
| 2 | What genetic factor determines the subtype (severity category) of alpha-thalassemia? | Số lượng gen alpha-globin bị mất/hỏng (HBA1/HBA2 copies) quyết định subtype: 1 gen → silent carrier, 2 gen → carrier, 3 gen → HbH, 4 gen → major. Từ `02_Alpha_Thalassemia_Fact_Sheet` (dòng 26, 116-132) |
| 3 | What is the basic human chromosome makeup described here? | 46 NST: 22 cặp autosome + 2 NST giới tính; nữ = XX, nam = XY. Từ `03_Mendelian_Inheritance_Lecture` (dòng 75) |
| 4 | What is the most common malignant brain tumour in children? | Medulloblastoma. Từ `04_Brain_Tumours_Factsheet` (dòng 69) |
| 5 | Why can brain tumours cause headaches and seizures? | Khối u phát triển làm tăng áp lực nội sọ bằng cách đẩy mô não hoặc chặn dòng chảy dịch não tủy (CSF), gây đau đầu và co giật. Từ `04_Brain_Tumours_Factsheet` (dòng 19) |

### Kết Quả Của Tôi

| # | Query | Top-1 Retrieved Chunk | Score | Relevant? | Top-1 đúng source? |
|---|-------|----------------------|-------|-----------|--------------------|
| 1 | In NIPT, what is the role of paternal DNA information? | `04_Brain_Tumours_Factsheet` — brain tumours overview | 0.2137 | Không — lấy nhầm doc về u não thay vì PrenatalGenome | ❌ Sai |
| 2 | What genetic factor determines the subtype of alpha-thalassemia? | `02_Alpha_Thalassemia_Fact_Sheet` — alpha-thalassemia overview | 0.1122 | **Có** — doc chính xác, chứa thông tin về 4 gen và 4 subtypes | ✅ Đúng |
| 3 | What is the basic human chromosome makeup? | `03_Mendelian_Inheritance_Lecture` — genetics lecture | 0.0976 | **Có** — doc chính xác, dòng 75 mô tả 46 NST | ✅ Đúng |
| 4 | What is the most common malignant brain tumour in children? | `04_Brain_Tumours_Factsheet` — brain tumours factsheet | 0.1961 | **Có** — doc chính xác, dòng 69 nói Medulloblastoma | ✅ Đúng |
| 5 | Why can brain tumours cause headaches and seizures? | `07_Cancer_Screening_Action_Guide` — cancer screening guide | 0.1432 | Không — lấy nhầm doc về cancer screening thay vì brain tumours | ❌ Sai |

**Bao nhiêu queries trả về chunk relevant trong top-1?** 3 / 5

**Chi tiết top-3 cho các query thất bại:**

| Query | Top-1 | Top-2 | Top-3 | Relevant trong top-3? |
|-------|-------|-------|-------|----------------------|
| Q1 (NIPT paternal DNA) | `04_Brain_Tumours` (0.2137) | `03_Mendelian` (0.1672) | `06_NIPT_Fact_Sheet` (0.0029) | Có — `06_NIPT` ở top-3 nhưng score rất thấp (0.003); file chính `01_PrenatalGenome` không xuất hiện |
| Q5 (brain tumour headaches) | `07_Cancer_Screening` (0.1432) | `03_Mendelian` (0.0857) | `05_Molecular_Dx` (0.0572) | Không — `04_Brain_Tumours` không xuất hiện trong top-3 |

> **Nhận xét:** Mock embedder (MD5 hash) cho kết quả retrieval ngẫu nhiên về mặt ngữ nghĩa. Tuy nhiên, 3/5 queries vẫn lấy đúng doc nhờ độ dài tài liệu tạo ra hash tình cờ có dot product cao hơn với query tương ứng. Q1 thất bại nặng nhất: file `01_PrenatalGenome` (~13,800 chars) không lọt top-3 dù chứa chính xác thông tin về paternal SNP analysis (dòng 69-77). Với real embedding model, semantic match sẽ giúp cả 5 queries lấy đúng doc.

---

## 7. What I Learned (5 điểm — Demo)

**Điều hay nhất tôi học được từ thành viên khác trong nhóm:**
> Việc so sánh RecursiveChunker và SentenceChunker trên cùng bộ tài liệu sinh học cho thấy không có strategy "one-size-fits-all". SentenceChunker hoạt động tốt hơn trên factsheet (câu ngắn, cấu trúc Q&A rõ ràng) còn RecursiveChunker tốt hơn trên whitepaper (paragraph dài, giải thích chi tiết quy trình kỹ thuật). Bài học quan trọng là nên chọn strategy dựa trên đặc điểm của từng loại tài liệu, không áp dụng cùng một strategy cho toàn bộ corpus.

**Điều hay nhất tôi học được từ nhóm khác (qua demo):**
> Các nhóm sử dụng domain khác (ví dụ: technical documentation, recipes) cho thấy metadata filtering cực kỳ quan trọng khi corpus đa dạng. Nhóm dùng metadata `category` để filter trước khi search giảm được noise đáng kể. Điều này khẳng định metadata schema design (topic, type, lang) mà nhóm mình chọn là hướng đi đúng, nhưng cần thực sự áp dụng filter trong queries thay vì search toàn bộ store.

**Nếu làm lại, tôi sẽ thay đổi gì trong data strategy?**
> Tôi sẽ: (1) Sử dụng real embedding model (all-MiniLM-L6-v2) thay vì mock để các khái niệm sinh học tương đồng (“paternal SNPs” ~ “fetal fraction estimation”) được ánh xạ gần nhau, (2) chunk documents bằng RecursiveChunker trước khi indexing — hiện tại doc Markdown (4,700 – 13,800 ký tự) được index nguyên file như 1 document duy nhất, chunking sẽ giữ tất cả thông tin ở dạng chunks nhỏ có ý nghĩa, và (3) tận dụng metadata filter `topic` — ví dụ query về NIPT nên filter `topic=prenatal_testing` để chỉ search trong 2 docs liên quan (file 01, 06) thay vì toàn bộ 7 docs.

### Failure Analysis (Ex 3.5)

**Query thất bại nặng nhất:** Q1 — "In NIPT, what is the role of paternal DNA information?"

**Retrieval thất bại vì 2 nguyên nhân độc lập:**

*Nguyên nhân chính — Mock embedder (không thể khắc phục bằng tuning):*
- Mock embedder tạo vector bằng MD5 hash của chuỗi ký tự, **hoàn toàn không encode ngữ nghĩa**. Query “paternal DNA information” và nội dung file 01 (dòng 69-77: “Single nucleotide polymorphisms (SNPs) inherited from the father were analyzed within the cfDNA to estimate fetal fraction…”) thuộc cùng topic, nhưng hash khác nhau → vector không tương quan → similarity score gần random.
- Hệ quả: Top-1 là `04_Brain_Tumours` (score=0.2137) — hoàn toàn không liên quan. File `01_PrenatalGenome` không lọt top-3.

*Nguyên nhân phụ — Pipeline chưa tối ưu (có thể cải thiện):*
- **Không chunk:** Mỗi file được index nguyên khố (~4,700–13,800 chars/doc). Thông tin về paternal SNP chỉ nằm ở mục "Estimation of Fetal DNA Fraction" (dòng 67-77) trong file 13,800 chars — bị “chìm” giữa hàng nghìn ký tự khác.
- **Metadata chưa tận dụng:** Không dùng `search_with_filter(topic=prenatal_testing)` để thu hẹp search space từ 7 xuống 2 docs (file 01 + 06).
- **Corpus đa sub-domain:** 7 docs trải từ genetics → oncology → prenatal → molecular diagnostics — với mock embedder, tất cả đều “trông giống nhau”.

**So sánh với Q2, Q3, Q4 (đã đúng):**
Q2 (alpha-thalassemia), Q3 (chromosome makeup), Q4 (medulloblastoma) lấy đúng doc nhờ **tình cờ** của MD5 hash tạo ra dot product cao hơn với doc tương ứng — không phải do hiểu ngữ nghĩa.

**Đề xuất cải thiện (theo thứ tự ưu tiên):**
1. **[Critical] Thay mock embedder** bằng real model (`all-MiniLM-L6-v2`) — fix quan trọng nhất, giúp “paternal DNA” gần “SNPs inherited from the father” trong vector space
2. **[High] Chunk trước khi index** — dùng RecursiveChunker (chunk_size=500) chia mỗi doc thành 10-30 chunks — chunk chứa "fetal fraction estimation" sẽ match tốt hơn cả file 13K
3. **[Medium] Áp dụng metadata filter** — `topic=prenatal_testing` cho NIPT, `topic=oncology` cho brain tumours
4. **[Nice-to-have] Thêm `sub_topic`** (ví dụ: `fetal_dna`, `thalassemia`, `brain_tumour`) để filter chính xác hơn

---

## Tự Đánh Giá

| Tiêu chí | Loại | Điểm tự đánh giá |
|----------|------|-------------------|
| Warm-up | Cá nhân | 5 / 5 |
| Document selection | Nhóm | 9 / 10 |
| Chunking strategy | Nhóm | 13 / 15 |
| My approach | Cá nhân | 9 / 10 |
| Similarity predictions | Cá nhân | 4 / 5 |
| Results | Cá nhân | 7 / 10 |
| Core implementation (tests) | Cá nhân | 30 / 30 |
| Demo | Nhóm | 4 / 5 |
| **Tổng** | | **81 / 100** |
