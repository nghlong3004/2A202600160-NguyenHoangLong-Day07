# PRENATAL GENOME

## Summary

Building on its legacy of genetic innovation, GENOMICA proudly introduces **PrenatalGenome**, the first non-invasive prenatal test (NIPT) that simultaneously screens for **chromosomal abnormalities** and **inherited** and **de novo** **single-gene disorders** (SGD).

While current NIPT methods screen for common and rare aneuploidies, segmental chromosomal imbalances, and syndromes associated with microdeletions or microduplications, **PrenatalGenome** goes further. It also analyzes circulating cell-free fetal DNA (cfDNA) in maternal blood for pathogenic and likely pathogenic mutations linked to single-gene disorders through **deep exome sequencing**, offering karyotype-level insights and more comprehensive risk assessment.

This white paper presents the analytical performance of the **PrenatalGenome** test, based on a retrospective analysis of **250 frozen plasma samples** from pregnant women who underwent traditional NIPT or non-invasive prenatal screening for de novo and/or inherited gene mutations. These results were verified through invasive prenatal diagnostic procedures, including amniocentesis and chorionic villus sampling.

The study demonstrates the test's ability to detect fetal aneuploidy, structural chromosomal abnormalities, and single-gene disorders from cfDNA in maternal blood with remarkable accuracy. Analytical **sensitivity and specificity exceeded 99%**, significantly reducing false positive and false negative results compared to traditional NIPT for fetal chromosomal abnormalities and NIPT for single-gene disorders (NIPT-SGD).

In conclusion, the **PrenatalGenome** test reliably identifies fetal chromosomal anomalies, microdeletion/microduplication syndromes, and genetic variants, including de novo mutations. This innovative method has the potential to redefine non-invasive prenatal testing and enable the detection of genetic conditions that current technologies cannot identify.

## Preclinical Validation Data and Performance Parameters

Current NIPT has mainly been limited to chromosomal disorders owing to the low resolution of existing screens, which is generally insufficient for identifying mutations causing single-gene disorders. Since SGDs contribute significantly to birth defects, it is important to enhance current prenatal screening methods to include these conditions.

There is a need for a comprehensive next-generation NIPT assay capable of simultaneously detecting chromosomal abnormalities and single-gene disorders from circulating cfDNA in maternal blood.

Existing NIPT methods for SGDs primarily focus on detection of de novo or paternally inherited mutations associated with common dominant monogenic disorders. However, these approaches are restricted to specific gene regions, limiting their ability to detect the wide range of sporadic mutations present in cfDNA.

Recently, a proof-of-concept approach leveraging deep exome sequencing has been proposed to enable the non-invasive detection of fetal de novo variants with high accuracy.

We developed a high-resolution, non-invasive prenatal screening approach utilizing **ultra-deep exome sequencing**. This technique examines the **whole fetal exome** through cfDNA from maternal blood samples. Since exonic variants account for a large fraction of disease-causing mutations in Mendelian disorders, this method provides a comprehensive tool for fetal genetic screening.

Additionally, invasive prenatal diagnostics currently recommend exome sequencing for pregnancies involving fetal structural anomalies due to significant diagnostic yield.

Our non-invasive fetal exome screening (niFES) test employs **ultra-deep sequencing** to deliver high-resolution results with increased detection rate and reduced false positive rates. By focusing on clinically relevant genes, it minimizes the identification of copy number variations (CNVs) with uncertain significance.

This approach enables simultaneous detection of chromosomal abnormalities, including aneuploidies, segmental imbalances, and microdeletions/duplications, as well as fetal **de novo variants**.

## Introduction

Fetal genetic diagnosis plays a critical role in prenatal care, and recent advancements in prenatal exome sequencing have demonstrated significant diagnostic improvements. However, due to the invasive nature of fetal sampling, its application remains limited to cases involving identifiable structural anomalies.

This limitation leaves many monogenic disorders undetected, as these conditions often do not manifest during the prenatal period. Consequently, a substantial number of neonates are born with severe or fatal genetic conditions.

Approximately 60% of severe postnatal monogenic diseases are dominant disorders, with the majority caused by **de novo** mutations.

The development of noninvasive prenatal testing (NIPT) using cell-free fetal DNA (cfDNA) extracted from maternal blood has revolutionized prenatal screening by enabling detection of fetal aneuploidy and structural chromosomal abnormalities.

However, its application has mainly been limited to chromosomal disorders owing to the low resolution available with existing screens, which are generally insufficient for identifying mutations causing single-gene disorders.

## Materials and Methods

### Study Design

This study retrospectively analyzed **250 frozen plasma samples** collected from pregnant women undergoing traditional NIPT and/or non-invasive prenatal screening for **de novo** and inherited single-gene disorders (NIPT-SGD). The test results were confirmed using invasive prenatal diagnostic methods, such as amniocentesis or chorionic villus sampling. Samples were collected between **10 and 22 weeks of gestation**.

Traditional NIPT was performed using the VeriSeq NIPT Solution v2 kit and VeriSeq NIPT Assay Software v2 (Illumina), following the manufacturer's instructions. This method provided an average sequencing depth of approximately **9.6 million reads per sample**.

The NIPT-SGD approach targeted **50 genes** associated with frequent monogenic disorders linked to severe health outcomes.

### Cell-Free DNA Extraction and Sequencing

Plasma was isolated from maternal blood collected in Streck tubes and processed promptly. Blood samples underwent initial centrifugation at **1,500 x g for 10 minutes at 4°C** to separate plasma from peripheral blood cells. The plasma fraction was transferred to polypropylene tubes and centrifuged again at **15,000 x g for 10 minutes at 4°C** to remove residual cells. The isolated plasma was stored at **-80°C** until further analysis.

Cell-free fetal DNA (cfDNA) was extracted from **1 mL** of maternal plasma using the QIAamp DSP Circulating NA Kit (Qiagen), following the manufacturer’s instructions.

Sequencing libraries were processed on the NextSeq 550 DX platform (Illumina) with ultra-deep exome sequencing achieving coverage of **>400x per sample**. Advanced technological approaches, including unique molecular indexing (UMI), were applied to minimize background noise and ensure accurate detection of low-level fetal DNA variants.

### Bioinformatic Data Analysis

A customized bioinformatics pipeline was developed for fetal copy number analysis, accurate variant calling and filtering, and error correction using unique molecular identifiers. The pipeline incorporated site-specific noise modeling and fetal fraction estimation, ensuring reliable identification of autosomal and sex chromosome aneuploidies, sub-chromosomal CNVs, and single-gene disorders.

### Estimation of Fetal DNA Fraction

The fetal fraction, meaning the proportion of cell-free DNA in a maternal blood sample that is of fetal origin, was calculated to confirm the presence of fetal DNA in maternal plasma. Single nucleotide polymorphisms (SNPs) inherited from the father were analyzed within the cfDNA to estimate fetal fraction and validate DNA detection accuracy.

The calculation was based on the following formula:

```text
Fetal DNA fraction = D_fetus / (D_mother + D_fetus)
```

where `D_mother` represents alleles shared between the mother and fetus, and `D_fetus` represents fetal-specific alleles.

### Performance Assessment for Detection of Fetal-Specific Variants

To evaluate the test’s ability to identify fetal-specific variants, a single nucleotide variant (SNV) was classified as fetal-specific if present in maternal plasma DNA or fetal genomic DNA (gDNA) but absent in maternal gDNA. **De novo** mutations, a specific type of fetal-specific variant, were defined as genetic changes not detected in either parental sample.

Fetal-specific variants were identified using SNV sites that were heterozygous in the fetus but homozygous in the mother, where the fetal allele represented the fetal-specific component. The performance of the test for detecting **de novo** mutations was evaluated based on its accuracy in identifying these fetal-specific variants.

The test results obtained from cfDNA samples were compared to paired fetal gDNA samples, considered the gold standard for validation.

Metrics used:

- Sensitivity
- Specificity
- Positive Predictive Value (PPV)
- Negative Predictive Value (NPV)

Where:

- TP (True Positives): variants identified in both plasma DNA and fetal gDNA
- FN (False Negatives): variants detected in fetal gDNA but absent in plasma DNA
- FP (False Positives): variants detected in plasma DNA but absent in fetal gDNA
- TN (True Negatives): variants correctly absent in both plasma and fetal gDNA

## Results

### Detection of Fetal Aneuploidy and Structural Chromosomal Abnormalities

A total of **250 frozen plasma samples** were retrospectively analyzed to assess the test’s effectiveness in detecting chromosomal abnormalities, including aneuploidies, segmental chromosomal imbalances, and microdeletion/microduplication syndromes.

Compared to traditional NIPT, the niFES test demonstrated superior sensitivity and specificity, with significant reductions in false positives and false negatives.

Overall sensitivity reached **100%**, and specificity improved to **91.2%**.

### Detection of De Novo Mutations

The niFES test also showed high sensitivity and specificity for detecting paternally inherited and **de novo** single nucleotide variants (SNVs), the largest category of clinically significant mutations after aneuploidies.

Among the **250 samples** analyzed, the test correctly identified all disease-causing SNVs, achieving **100% sensitivity** and **100% specificity**.

Notably, the niFES method resulted in zero false negatives and significantly fewer false positives compared to NIPT-SGD.

## Conclusion

We have developed a groundbreaking platform for the simultaneous non-invasive prenatal detection of chromosomal abnormalities and monogenic diseases with analytical sensitivity and specificity exceeding **99.9%**.

The innovative niFES test, based on deep exome sequencing, offers a fast, comprehensive, and accurate method to identify a wide range of chromosomal and genetic disorders without imposing risks on the fetus or mother.

Our results demonstrate that niFES can reliably detect fetal chromosomal abnormalities, including rare aneuploidies, structural aberrations, and single nucleotide variants (SNVs). The platform also effectively identifies **de novo** and inherited variants.

By integrating niFES into routine prenatal care alongside fetal ultrasonography, clinicians could improve early detection rates of genetic disorders, reduce the number of invasive diagnostic procedures, and enable timely interventions.

This paradigm shift in prenatal screening offers a clearer and more complete picture of the genetic risks affecting pregnancies.

## References

1. Mellis R, Oprych K, Scotchman E, Hill M, Chitty LS. Diagnostic yield of exome sequencing for prenatal diagnosis of fetal structural anomalies: a systematic review and meta-analysis. Prenat Diagn. 2022.
2. Miceikaite I, Fagerberg C, Brasch-Andersen C, et al. Comprehensive prenatal diagnostics: exome versus genome sequencing. Prenat Diagn. 2023.
3. Chitty LS. Advances in the prenatal diagnosis of monogenic disorders. Prenat Diagn. 2018.
4. Yang Y, et al. Molecular findings among patients referred for clinical whole-exome sequencing. JAMA. 2014.
5. Norton ME. Circulating cell-free DNA and screening for trisomies. N Engl J Med. 2022.
6. Bianchi DW, Parker RL, Wentworth J, et al. DNA sequencing versus standard prenatal aneuploidy screening. N Engl J Med. 2014.
7. Scotchman E, Chandler NJ, Mellis R, Chitty LS. Noninvasive Prenatal Diagnosis of Single-Gene Diseases: The Next Frontier. Clin Chem. 2020.
8. Zhang J, Li J, Saucier JB, et al. Non-invasive prenatal sequencing for multiple Mendelian monogenic disorders using circulating cell-free fetal DNA. Nat Med. 2019.
9. Brand H, Whelan CW, Duyzend M, et al. High-resolution and noninvasive fetal exome screening. N Engl J Med. 2023.
10. Miceikaitė I, Hao Q, Brasch-Andersen C, et al. Comprehensive Noninvasive Fetal Screening by Deep Trio-Exome Sequencing. N Engl J Med. 2023.
11. Van den Veyver IB, Chandler N, Wilkins-Haug LE, et al. Updated position statement on the use of genome-wide sequencing for prenatal diagnosis. Prenat Diagn. 2022.
12. Wright CF, Campbell P, Eberhardt RY, et al. Genomic diagnosis of rare pediatric disease in the United Kingdom and Ireland. N Engl J Med. 2023.
13. Lo YM, Chan KC, Sun H, et al. Maternal plasma DNA sequencing reveals the genome-wide genetic and mutational profile of the fetus. Sci Transl Med. 2010.
14. Dan S, Yuan Y, Wang Y, et al. Non-Invasive Prenatal Diagnosis of Lethal Skeletal Dysplasia by Targeted Capture Sequencing of Maternal Plasma. PLoS One. 2016.
15. Fiorentino et al. The clinical utility of genome-wide noninvasive prenatal screening. Prenat Diagn. 2017.
