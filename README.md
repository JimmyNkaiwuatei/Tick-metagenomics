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

# Sequence retrieval and filtration
The raw reads from tick pools tested will be easily retrieved from https://idseq.net/ under the project name “KWS ticks” and in National Center for Biotechnology Information (NCBI) biosample database with the ID: SUB11147880.

## Generating host removal databases
The process for generating host removal databases is as follows.

[Build the STAR database:](https://github.com/chanzuckerberg/idseq-dag/blob/master/idseq_dag/steps/generate_host_genome.py)

STAR
--runThreadN {cpu_count}
--runMode genomeGenerate
--sjdbGTFfile {gtf_command_part}*  *
--genomeDir {star_genome_part_dir}
--genomeFastaFiles {fasta_file_list[i]}* *

note: the --sjdbGTFfile option and .gtf file is only used for database generation in cases where this file is available.

ERCCs in the STAR database: The External RNA Control Consortium (ERCC) developed a set of spike-in controls that can be used to measure sensitivity, accuracy, and biases in RNA-seq experiments as well as to derive standard curves for quantifying the abundance of transcripts (manuscript available [here](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3166838/)). Quantification of ERCCs is done in the first host removal pipeline step, using STAR. The ERCC reference sequences are appended to the host reference .fasta file and a .gtf file is used to enable quantification of ERCC abundance via the built-in STAR gene-count capabilities.

[Build the Bowtie2 database:](https://github.com/chanzuckerberg/idseq-dag/blob/master/idseq_dag/steps/generate_host_genome.py)

bowtie2-build {fasta_file} {host_name}

Build the GSNAP database for final human removal: This was generated one-time only, using the following command:

# gsnap version 2017-08-25
gmap_build 
-d hg38_pantro5_k16 
-D ./gsnap 
-k 16 
hg38_pantro5.fa

Pipeline Input

[RunValidateInput](https://github.com/chanzuckerberg/idseq-dag/blob/master/idseq_dag/steps/run_validate_input.py)

Input: raw .fastq files

Process: Validates that the input files are .fastq format and truncates to 75 million fragments (specifically, 75 million reads for single-end libraries or 150 million reads for paired-end libraries). The validation process counts the number of sequences that fall in specified length buckets, which will inform the parameters used downstream for initial host removal by STAR.

Output:

    validate_input1.fastq
    validate_input2.fastq
    validate_input_summary.json: {"<50": 0, "50-500": 93621550, "500-10000": 0, "10000+": 0}
    
## Build the star database
    STAR
--runThreadN {cpu_count}
--runMode genomeGenerate
--sjdbGTFfile {gtf_command_part}* *
--genomeDir {star_genome_part_dir}
--genomeFastaFiles {fasta_file_list[i]}* *

## Build Bowtie2 database
Bowtie2-build {fasta_file} {host_name}

Build the GSNAP database for final human removal: This will be generated one-time only, using the following command:
gsnap version 2017-08-25

gmap_build
-d hg38_pantro5_k16
-D ./gsnap
-k 16
hg38_pantro5.fa
