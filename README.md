# SBCapstone_CRISPRScreens

Identify features that predict essential and non-essential genes. 

## Background

The CRISPR-Cas system has been revolutionary for science. The ability to knockout specific genes and then associate that gene with a
phenotype is vital for functional studies. Many groups have turned to large-scale 'pooled' screens. In this case, the researcher
attempts to knockout all (or at least most) protein coding genes in a cell. One phenotype that is easily assessed is viability. That is,
if I knock this gene out, does the cell survive. This analysis is typically indirect. For most screens, multiple guides (the sequence
that is used to target a specific gene) are used. Experiments are performed such that, on average, each cell only gets one guide. 
Typically, multiple guides per gene are used. The logic is as follows:

* Get the count of each guide prior to putting the guides in the cell population. 
* Transduce the guides into the cell population
    + one guide per cell (on average)
    + the guides stably integrate into the genome
* Count the guides again over a period of time, or after a treatments  
    + guides that knock out essential genes should deplete

Caveats with this approach:
* Guides counts can change within a library due to random chance. Therefore, the key is to identify changes that are significantly
different from random chance. (references below)
* Most approaches use some sort of statistical test to assess this. 
* Most researchers put up with a high false negative and false positive rate as this is just used as a screen. 

Goal of this project:
* can we identify features that predict whether a gene is 'essential' (assuming no treatment, just the KO and growth). 

Things to consider for this model building:

Feature Engineering:

We can look both at some transcript intrinsic information to select features. This will likely be closer to the actual biology. Candidate features are:

* Native features 
    + transcript length (this is the length of the primary mRNA used in this study)
    + gene length (this is the length of the introns + exons) [Need to get]
    + total number of exons (only exons go into making mRNAs.)
    + expression level [Need to get]
        + note: this will be cell type specific so we would need to incorporate cell type in a fully robust model. 
    + isoform number
    + constraint score (this is a proxy for how much nature variation is in a gene) [Need to get]
    + Targeted Exon 

* Derived features:
    + transcript length/exon number
    + gene length/exon number
    + gene length/transcript length
    + expression level quartiles


We can also look at aspects of the guides. This is likely less about the biology of the gene being knocked out and more about technical aspects of the library (meaning, things that may lead to artifacts).
For each gene, there is typically 4 guides, so we may need to consider some aggregate information. 

* Native (per guide) features
    + PAM sequence used (per guide)
    + strand used (per guide)
* Aggregates (per gene)
    + mean starting count (per gene): standardized
    + mean final count (per gene): standardized
    + variance across guides (per gene): standardized
    + std dev across guides (per gene): standardized
    + mean quartile starting count (per gene): standardized
    + mean quartile final count (per gene): standardized.

## Challenges

It is very difficult to obtain raw data (that is the raw counts) for various experiments. Therefore, the data we have will be 
relatively limited. 

### Proposal

Use data from Sanson et al., 2018 to identify features and build a model to predict gene essentiality. 

Notes:
Gene essentiality is likely cell type specific, at least in many cases. It is unclear how well this model will 
transfer. 

## Notebook breadcrumbs

### Processing Gene and Transcript information

GetTranscriptInfo.ipynb: Notebook for processing some transcript information. 

GetIsoformInformation.ipynb: Notbook for getting isoform counts. 

AnnotateGenes.ipynb: Annotate genes with essentiality status (essential, non-essential or unknown) 

GetGeneLength.ipynb: Get the full length of each gene.

CalcGeneExpression_A375_GSE249290.ipyn: estimated mean gene expression across several conditions for A375 cell line. 

### Data Processing
Brunello_count_library.ipynb: This is the main notebook that was used for initial EDA analysis. This integrates both transcript information and count information from the brunello library.


### Abandoned Dataset
SARs_Datsets.ipynb: Initial notebook for two datasets obtained from BioGrids. Realize these are not usable datasets as this is only processed data, not raw counts. 

# Task list

As of 2024-10-29, keeping tasks list as github issues. 


## Info from papers

|                              |  Sanson et al., 2018              |    Israeli et al., 2022          | 
|------------------------------|-----------------------------------|----------------------------------|
| **Main question of paper**   | | Developing a new CRISPRko library that can more effectively distinguish between essential and non-essential genes.  Note: they also look at CRISPRi and CRISPRa libraries, but we won't focus on that here. | Understanding cellular factors involved in SARS-CoV-2 infection. (original variant, as well as Alpha and Beta variants) | Understanding of cellular factors involved in SARS-CoV-2 infection.  |
|**Cell line used**            | A375 melanoma cell lines          | Calu-3  (human lung cells)                                            |
|**Cell Numbers**              |                                   | 400 M                                                                     |
|**Library Used**              | Brunello                          | Brunello                                                             |
|**Guide Info**                | 77,441 sgRNA (1000 non-target controls) | 76,441 sgRNAs (1000 non-target controls)                            |
|**Avg guide per gene**        | 4                                 | 4                                                                          |
|**Replicates**                | 2-3                               | 2                                                                          |
|**Sequence**                  |https://www.ncbi.nlm.nih.gov/sra/PRJNA508200 |                                                                  |




## Data files fro BioGRID ORCs and/or Papers

|                     | File                                    | Description                                             |
|---------------------|-----------------------------------------|---------------------------------------------------------|
| Israeli et al, 2022 | BIOGRID-ORCS-5622-1.1.16.screen.tab.txt | Table from BioGrid ORCs                                 |
|                     | PMID35469023-SuppData3.xlsx             | Supplemental table 3 MAGeCK output                      |
|                     | GSE197962_sgRNA_counts.txt              | Guide counts from                                       |
|                     | broadgpp-brunello-library-contents.txt  | guide-gene mapping                                     |
| Sanson et al., 2018 | 41467_2018_7901_MOESMA_ESM.xlsx         | Supplmentary table 1 (raw counts for KO)                |
|                     | broadgpp-brunello-library-contents.txt | library info from AddGene                               |
|                     | brunello_plasmid_counts                 | Initial info on plasmid and guide counts in A375 screen |

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

Focused on accurate modeling of read count distribution in CRISPR screens. Compared abundance just from the culturing process- found that before/after ratios for control guides are not asymmetric in KO screens- influenced by cell proliferation and width of sgRNA abundance distribution- so a skew normal distribution is use to estimate the null distribution.

## Things that may impact the data

-   Guide counts general exhibit overdispersion

-   Sequecing depth

-   Gene Expression

-   Selection approach

-   Guide efficiency