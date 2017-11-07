# ChIP-seq Hands-on
### Introduction
#### Goal
The aim is to :
  * have an understanding of the nature of ChIP-Seq data
  * perform a complete analysis workflow including quality check (QC), read mapping, visualization in a genome browser and peak-calling. Use command line and open source software for each each step of the workflow and feel the complexity of the task
  * have an overview of possible downstream analyses
  * perform a motif analysis with online web programs

#### Summary
This training gives an introduction to ChIP-seq data analysis, covering the processing steps starting from the reads to the peaks. Among all possible downstream analyses, the practical aspect will focus on motif analyses. A particular emphasis will be put on deciding which downstream analyses to perform depending on the biological question. This training does not cover all methods available today. It does not aim at bringing users to a professional NGS analyst level but provides enough information to allow biologists understand what DNA sequencing practically is and to communicate with NGS experts for more in-depth needs.

#### Dataset description
For this training, we will use a dataset produced by Myers et al (see on the right of the page for details on the publication) involved in the regulation of gene expression under anaerobic conditions in bacteria. We will focus on one factor: FNR.

## Downloading ChIP-seq reads from NCBI
**Goal**: Identify the dataset corresponding to the studied article and retrieve the data (reads as FASTQ file) corresponding to one experiment and the control.  
**Related VIB training**: Downloading NGS data from NCBI

### 1 - Obtaining an identifier for a chosen dataset
Within an article of interest, search for a sentence mentioning the deposition of the data in a database. Here, the following sentence can be found at the end of the Materials and Methods section:
*"All genome-wide data from this publication have been deposited in NCBI’s Gene Expression Omnibus (**GSE41195**)."*
We will thus use the **GSE41195** identifier to retrieve the dataset from the **NCBI GEO** (Gene Expression Omnibus) database.

NGS datasets are (usually) made freely accessible for other scientists, by depositing these datasets into specialized databanks. Sequence Read Archive (SRA) located in USA hosted by NCBI, and its european equivalent European Nucleotide Archive (ENA) located in England hosted by EBI both contains **raw reads**.

Functional genomic datasets (transcriptomics, genome-wide binding such as ChIP-seq,...) are deposited in the databases Gene Expression Omnibus (GEO) or its European equivalent ArrayExpress.

### 2 - Accessing GSE41195 from GEO
1.  The GEO database hosts processed data files and many details related to the experiments. The SRA (Sequence Read Archive) stores the actual raw sequence data.
2. Search in google **GSE41195**. Click on the first link to directly access the correct page on the GEO database.
3. This GEO entry is a mixture of expression analysis and chip-seq. At the bottom of the page, click on the subseries related to the chip-seq datasets. (this subseries has its own identifier: **GSE41187**).
4. From this page, we will focus on the experiment **FNR IP ChIP-seq Anaerobic A**. At the bottom of the page, click on the link "GSM1010219 - FNR IP ChIP-seq Anaerobic A".
5. In the new page, go to the bottom to find the SRA identifier. This is the identifier of the raw dataset stored in the SRA database.
6. Copy the identifier **SRX189773** (do not click on the link that would take you to the SRA database, see below why)

### 3 - Downloading FASTQ file from the ENA database
Although direct access to the SRA database at the NCBI is doable, SRA does not store the sequence FASTQ format. In practice, it's simpler (and quicker!!) to download datasets from the ENA database (European Nucleotide Archive) hosted by EBI (European Bioinformatics Institute) in UK. ENA encompasses the data from SRA.

1. Go to the EBI website. Paste your SRA identifier (SRX189773) and click on the button "search".
2. Click on the first result. On the next page, there is a link to the FASTQ file. For efficiency, this file has already been downloaded and is available in the "data" folder (SRR576933.fastq)

## Quality control of the reads and statistics
**Goal**: Get some basic information on the data (read length, number of reads, global quality of dataset)  
**Related VIB training**: Quality control of NGS data

### 1 - Getting the FASTQC report
Before you analyze the data, it is crucial to check the quality of the data. We will use the standard tool for checking the quality of data generated on the Illumina platform: FASTQC. Now is the time to gather your courage and face the terminal. Don't worry, it's not as scary as it seems ! Below, the "$" sign is to indicate what you must type in the command line, but don't type the "$".

1. Open the terminal and go to the course directory.
```bash
cd /home/bits/NGS/ChIPSeq
```
2. First check the help page of the program to see its usage and parameters.
```bash
srun fastqc --help
```
3. Launch the FASTQC program on the experiment file (SRR576933.fastq)
```bash
srun fastqc SRR576933.fastq
```
4. Wait until the analysis is finished. Check the files output by the program.
```bash
ls
SRR576933_fastqc  SRR576933_fastqc.zip	SRR576933.fastq
```
6. Access the result
```bash
cd SRR576933_fastqc
ls
fastqc_data.txt  fastqc_report.html  Icons  Images  summary.txt
```
7. Open the HTML file fastqc_report.html in Firefox.

### 2 - Organism length
Knowing your organism size is important to evaluate if your dataset has sufficient coverage to continue your analyses. For the human genome (3 Gb), we usually aim at least 10 Million reads.

1. Go to the NCBI Genome website, and search for the organism **Escherichia coli**
2. Click on the **Escherichia coli str. K-12 substr. MG1655** to access statistics on this genome.

## Mapping the reads with Bowtie
**Goal**: Obtain the coordinates of each read on the reference genome.  
**Related VIB training**: Mapping of NGS data

### 1 - Choosing a mapping program
There are multiple programs to perform the mapping step. For reads produced by an Illumina machine for ChIP-seq, the currently "standard" programs are BWA and Bowtie (versions 1 and 2), and STAR is getting popular. We will use **Bowtie version 1.1.1** for this exercise, as this program remains effective for short reads (< 50bp).

### 2 - Prepare the index file
1. Try out bowtie
```bash
bowtie-1.1.1
```
This prints the help of the program. However, this is a bit difficult to read ! If you need to know more about the program, it's easier to directly check the manual on the website (if it's up-to-date !)
2. bowtie needs the reference genome to align each read on it. This genome needs to be in a specific format (=index) for bowtie to be able to use it. Several pre-built indexes are available to download on the bowtie webpage, but our genome is not there. You will need to make this index file.
3. To make the index file, you will need the complete genome, in FASTA format. It has already been downloaded to gain time (Escherichia_coli_K12.fasta in the course folder) (The genome was downloaded from the NCBI). Note that we will not work with the lastest version (NC_000913.3) but the previous one (NC_000913.2), because the available tools for visualization have not been updated yet to the latest version. This will not affect our results.
4. Build the index for bowtie
```bash
bowtie-1.1.1-build Escherichia_coli_K12.fasta Escherichia_coli_K12
```

### 3 - Mapping the experiment
1. Let's see the parameters of bowtie before launching the mapping:
  * Escherichia_coli_K12 is the name of our genome index file
  * Number of mismatches for SOAP-like alignment policy (-v): to 2, which will allow two mismatches anywhere in the read, when aligning the read to the genome sequence.
  * Suppress all alignments for a read if more than n reportable alignments exist (-m): to 1, which will exclude the reads that do not map uniquely to the genome.
  * -q indicates the input file is in FASTQ format. SRR576933.fastq is the name of our FASTQ file.
  * -3 will trim x base from the end of the read. As our last position is of low quality, we'll trim 1 base.
  * -S will output the result in SAM format
  * 2> SRR576933.out will output some statistics about the mapping in the file SRR576933.out
```bash  
$ bowtie-1.1.1 Escherichia_coli_K12 -q SRR576933.fastq  -v 2 -m 1 -3 1 -S 2> SRR576933.out > SRR576933.sam
```
2. This should take few minutes as we work with a small genome. For the human genome, we would need either more time, or a dedicated server.

### 4 - Mapping the control
1. Repeat the steps above (step 3) for the file SRR576938.fastq.

## Bonus: checking two ENCODE quality metrics
**Goal**: This optional exercice aims at calculating the NSC and RSC ENCODE quality metrics. These metrics allow to classify the datasets (after mapping, contrary to FASTC that works on raw reads) in regards to the NSC and RSC values observed in the ENCODE datasets (see ENCODE guidelines)

### 1 - PhantomPeakQualTools
Warning : this exercice is new and is tested for the first time in a classroom context. Let us know if you encounter any issue.

The SAM format correspond to large text files, that can be compressed ("zipped") into BAM format. The BAM format are usually sorted and indexed for fast access to the data it contains.

1. convert the SAM file into a sorted BAM file
```bash
samtools-0.1.19 view -bS SRR576933.sam  > SRR576933.bam
```
2. convert the BAM file into TagAlign format, specific to the program that calculates the quality metrics
```bash
samtools-0.1.19 view -F 0x0204 -o - SRR576933.bam | awk 'BEGIN{OFS="\t"}{if (and($2,16) > 0) {print $3,($4-1),($4-1+length($10)),"N","1000","-"} else {print $3,($4-1),($4-1+length($10)),"N","1000","+"} }' | gzip -c > SRR576933_experiment.tagAlign.gz
```
3. Run phantompeakqualtools
```bash
Rscript /usr/bin/tools/phantompeakqualtools/run_spp.R -c=SRR576933_experiment.tagAlign.gz  -savp -out=SRR576933_experiment_phantompeaks
```

## Peak calling with MACS
**Goal**: Define the peaks, i.e. the region with a high density of reads, where the studied factor was bound

### 1 - Choosing a peak-calling program
There are multiple programs to perform the peak-calling step. Some are more directed towards histone marks (broad peaks) while others are specific to narrow peaks (transcription factors). Here we will use MACS version 1.4.2 because it's known to produce generally good results, and it is well-maintained by the developper. A new version (MACS2) is being developped, but still in testing phase so we will not use it today.

### 2 - Calling the peaks
1. Try out MACS
```bash
macs14
```
This prints the help of the program.
2. Let's see the parameters of MACS before launching the mapping:
  * ChIP-seq tag file (-t) is the name of our experiment (treatment) mapped read file SRR576933.sam
  * ChIP-seq control file (-c) is the name of our input (control) mapped read file SRR576938.sam
  * --format SAM indicates the input file are in SAM format. Other formats can be specified (BAM,BED...)
  * --gsize Effective genome size: this is the size of the genome considered "usable" for peak calling. This value is given by the MACS developpers on their website. It is smaller than the complete genome because many regions are excluded (telomeres, highly repeated regions...). The default value is for human (2700000000.0), so we need to chnage it. As the value for E. coli is not provided, we will take the complete genome size 4639675.
  * --name provides a prefix for the output files. We set this to macs14, but it could be any name.
  * --bw The bandwidth is the size of the fragment extracted from the gel electrophoresis or expected from sonication. By default, this value is 300bp. Usually, this value is indicated in the Methods section of publications. In the studied publication, a sentence mentions "400bp fragments (FNR libraries)". We thus set this value to 400.
  * --keep-dup specifies how MACS should treat the reads that are located at the exact same location (duplicates). The manual specifies that keeping only 1 representative of these "stacks" of reads is giving the best results.
  * --bdg --single-profile will output a file in BEDGRAPH format to visualize the peak profiles in a genome browser. There will be one file for the treatment, and one for the control.
  * --diag is optional and increases the running time. It tests the saturation of the dataset, and gives an idea of how many peaks are found with subsets of the initial dataset.
  * &> MACS.out will output the verbosity (=information) in the file MACS.out
```bash
$ macs14 -t SRR576933.sam -c SRR576938.sam --format SAM  --gsize 4639675 --name "macs14"  --bw 400 --keep-dup 1 --bdg --single-profile --diag &> MACS.out
```
3. This should take about 10 minutes, mainly because of the --diag and the --bdg options. Without, the program runs much faster.

### 3 - Analyzing the MACS results