# SBCapstone_CRISPRScreens

Goal of Capstone: investigate sensitivity and power of lethality based CRISPR screens. Focus on viability aspects of the screens.

## Current issues

Concerns about using data from the Israeli et al paper and the Cunha et al paper:

-   Lack off access to raw count data from both studies

-   Size imbalance:

    -   Israeli et al is a whole genome (roughly 20K genes targeted) and Cunha et al only targets 833 genes

-   The studies use different guides to target the genes (and only Israeli uses a publicly available library).

### Proposal

Switch to characterizing the guides and performance in the Sanson paper.

## Info from papers

|                              |  Sanson et al., 2018              |    Israeli et al., 2022          |    Cunha et al., 2023                     |
|------------------------------|-----------------------------------|----------------------------------|-------------------------------------------|
| **Main question of paper**   | | Developing a new CRISPRko library that can more effectively distinguish between essential and non-essential genes.  Note: they also look at CRISPRi and CRISPRa libraries, but we won't focus on that here. | Understanding cellular factors involved in SARS-CoV-2 infection. (original variant, as well as Alpha and Beta variants) | Understanding of cellular factors involved in SARS-CoV-2 infection. |   Cellular factors involved in SARS-CoV-3 infection |
|**Cell line used**            | A375 melanoma cell lines          | Calu-3  (human lung cells)        | Calu-3                                      |
|**Cell Numbers**              |                                   | 400 M                             | 50M                                         |
|**Library Used**              | Brunello                          | Brunello                          | Custom                                      |
|**Guide Info**                | 77,441 sgRNA (1000 non-target controls) | 76,441 sgRNAs (1000 non-target controls)| N/A                             |
|**Avg guide per gene**        | 4                                 | 4                                 |                                             |
|**Replicates**                | 2-3                               | 2                                 | 2                                           |
|**Sequence**                  |https://www.ncbi.nlm.nih.gov/sra/PRJNA508200 |                         |                                             |




## Data files fro BioGRID ORCs and/or Papers

+---------------------+-----------------------------------------+---------------------------------------------------------+
|                     | File                                    | Description                                             |
+=====================+=========================================+=========================================================+
| Cunha et al., 2023  | BIOGRID-ORCS-5622-1.1.16.screen.tab.txt | Table from BioGrid ORCs                                 |
+---------------------+-----------------------------------------+---------------------------------------------------------+
|                     | jv.01276-23-s0004.xlsx                  | Supplemental table 4 (plasmid vs. d0 gene)\             |
|                     |                                         | MAGeCK output                                           |
+---------------------+-----------------------------------------+---------------------------------------------------------+
| Israeli et al, 2022 | BIOGRID-ORCS-5622-1.1.16.screen.tab.txt | Table from BioGrid ORCs                                 |
+---------------------+-----------------------------------------+---------------------------------------------------------+
|                     | PMID35469023-SuppData3.xlsx             | Supplemental table 3                                    |
|                     |                                         |                                                         |
|                     |                                         | MAGeCK output                                           |
+---------------------+-----------------------------------------+---------------------------------------------------------+
|                     | GSE197962_sgRNA_counts.txt              | Guide counts from                                       |
+---------------------+-----------------------------------------+---------------------------------------------------------+
|                     | broadgpp-brunello-library-contents.txt  | guide-gene mapping                                      |
+---------------------+-----------------------------------------+---------------------------------------------------------+
|                     | brunello_plasmid_counts                 | Initial info on plasmid and guide counts in A375 screen |
+---------------------+-----------------------------------------+---------------------------------------------------------+
| Sanson et al., 2018 | 41467_2018_7901_MOESMA_ESM.xlsx         | Supplmentary table 1 (raw counts for KO)                |
+---------------------+-----------------------------------------+---------------------------------------------------------+

## Algorithms and approaches for analyzing CRISPR KO libraries

Mostly from Zhao et al, 2022

### Brief Description of MAGeCK

from Li et al., 2014 (original paper describing MAGeck)

1.  Read counts from different samples are median normalized to adjust for the effect of library sizes and read count distributions.

2.  The variance of read counts is estimated by sharing information across features

3.  A negative binomial model is used to test whether sgRNA abundance differs significantly between treatments and controls

    1.  Used sgRNAs based on P-values calculated from the negative binomial model

    2.  Used a modified robust ranking aggregation (RRA) algorithm to identify positively and negatively selected genes- alphaRRA

        1.  alphaRRA assumes that if a gene has no effect on select, then sgRNAs targeting this gene should be uniformly distributed across the ranked list of all sgRNAs.

4.  alphaRRA ranks genes by comparing the skew in rankings to the uniform null model and prioritizes gens whose sgRNA rankings are consistently higher than expected.

5.  alphaRRA calculates the skew by permutation

### ScreenBEAM

Use linear model equivalent to a students t-statistics to test significance of the coefficient.

### BAGEL

Used a predefined list of essential and non-essential genes. Estimated the sgRNA fold change for these genes using a kernel density estimate function. Then BAGEL evaluates the probability that the abundance changes of all sgRNAs for one gene were extracted from the essential gene or non-essential gene distribution.

### PBNPA

Computed a p-value at the gene level by the permutation test, with no-distribution assumptions.

### JACKS

Bayesian method: model sgRNA efficiencies by obtaining information from multiple screens utilizing the same sgRNA library design.

### gscreend

Focused on accurate modeling of read count distribution in SCRISPR screens. Compared abundance just from the culturing process- found that before/after ratios for control guides are not asymmetric in KO screens- influenced by cell proliferation and width of sgRNA abundance distribution- so a skew normal distribution is use to estimate the null distribution.

## Things that may impact the data

-   Guide counts general exhibit overdispersion

-   Sequecing depth

-   Gene Expression

-   Selection approach

-   Guide efficiency