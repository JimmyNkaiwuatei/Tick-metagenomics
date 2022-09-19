# Introduction

Metagenomic analysis of the tick samples will be performed using IDseq tool. The tool will be used to filter out host sequences and subsequently detect the pathogens present.
Also, VirldAl tool will be used for viruses detection.
The raw reads from tick pools tested will be easily retrieved from https://idseq.net/ under the project name “KWS ticks” and in National Center for Biotechnology Information (NCBI) biosample database with the ID: SUB11147880.
IDseq pipeline will be used to implement this project as follows:

1. Initial host filtration using STAR
2. Trim sequencing adapters using Trimmomatic
3. Quality filter using PriceSeq
4. Identify duplicate reads using idseq-dedup (note: prior to pipeline version 6.0, IDseq used CD-HIT-DUP for duplicate identification)
5. Filter out low complexity sequences using LZW
6. Filter out remaining host sequences using Bowtie2
7. Subsampling to 1 million fragments (reads/read-pairs) if > 1M remain after step (6)
8. Filter out human sequences, regardless of host (using STAR, Bowtie2, and GSNAP).

# Work plan
## Steps
## Upload
Upload FASTA/FASTQ files from your computer, or import from Illumina's Basespace.## 2. Find matches in NCBI.
The pipeline searches against the nucleotide and protein databases in NCBI for plant viruses, parasitic worms, bacteria, etc.

## 3. View Per-Sample Taxon Counts.
Check out the auto-generated results page to see the microorganisms found in each sample.

## 4. Analyze & Visualize.
Generate heatmaps, phylogenetic trees, or quality control charts to help draw conclusions across your samples.

## 5. Push button pipelines & results.
Discover tool potentially identifies infectious organisms with an easy-to-use GUI—no coding required.

## 6. Global pathogen datasets.
Explore datasets from researchers across the globe.
## 7. Downstream visualizations.
Visualize your samples in aggregate using the heatmap and phylogenetic tree modules.


![Image](https://user-images.githubusercontent.com/108969618/190356869-0ab4c3fb-fdfe-4572-bc7e-5fd8535681ad.png)


## Methods and tools

The IDseq pipeline is an open source tool that incorporates the following steps for validation:
i) Human and barcode adaptor removal.
ii) Quality assessment.
iii) Alignment.
IV) Assembly, and taxonomic identification using the National Center for Biotechnology Information (NCBI) nucleotide and protein databases.

Pathogens will be considered as significant when ≥ 2 nucleotide reads or contigs are verified in the final output. Relative abundances will be calculated in samples with bacterial reads ≥ 102 and expressed as taxon read count/total bacteria read count.
Reads and contigs from viruses, parasites, and tick-borne bacteria will be manually reviewed with the taxon identity confirmed using BLAST ([Altschul et al., 1990](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#B1)).

Viral reads will also be recovered using the VirIdAl pipeline ([Budkina et al., 2021](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#B4)). This pipeline will merge paired-end reads to increase sequence length after which sequences will be adapter-trimmed and quality-filtered using fastp ([Chen et al., 2018](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#B6)).
Sequences with a complexity of <30%, with an average PHRED quality score of <20, and with <36 bases will be removed. Host sequences will also be removed with Bowtie2 ([Langmead and Salzberg, 2012](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#B31)), using the most closely related available tick reference genome (Rhipicephalus sanguineus, ASM1333969v1).
Data will then be clustered using vsearch ([Rognes et al., 2016](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#B56)) to recover “centroid” sequences formed with a 0.9 identity threshold.
These sequences will then enter a two-step alignment process:
i) First, sequences will be blasted against both viral NT and NR databases at high sensitivity (e-value 10–3) to give “virus-like” sequence matches.
ii) The sequences will then be passed to a second step alignment to the complete NCBI NT and NR databases (e-value 10–10) for final classification.

CZ-ID will be used for the initial workup while VirIdAl will be used on samples with detectable virus signal. Outputs from both pipelines will be combined for producing final contigs, following duplicate removal.

Virus genome assemblies and Sanger sequencing data analyses will be carried out using Geneious Prime version 2022.0.2[1](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#footnote1).
Nucleotide and deduced amino acid sequence alignments and pairwise comparisons will be generated using CLUSTAL W ([Thompson et al., 1994](https://www.frontiersin.org/articles/10.3389/fmicb.2022.932224/full#B64)).
MEGA11 will be used for estimating the optimal substitution model on individual alignments and to infer evolutionary history according to the Bayesian information criteria.

## cs-id/IDseq pipeline:
https://github.com/chanzuckerberg/czid-pipeline

## IDseq workflows; latest version:
https://github.com/chanzuckerberg/idseq-workflows
## Python workflow
https://github.com/chanzuckerberg/idseq-workflows
## VirldAl- viral sequences search tool:
 https://github.com/budkina/VirIdAl.
