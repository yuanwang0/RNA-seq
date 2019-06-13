# RNA-seq

***
## I. Overview

Here we demonstrate how to convert raw `.fastq` files into quantified expression files for RNA-seq data.

The entire pipeline can be broken down into two major parts:
#### Step 1: Align reads into a reference genome

I/O       |   Format   | Description
--------- | ---------- | -----------------------------
Input     |   FASTQ    |  Unaligned raw reads
Input     |   FASTA    |  Reference genome/transcrpts
Input     |    GTF     |  Gene Annotations
Output    |    BAM     |  Aligned reads

#### Step 2: Quantify and normalize expression levels by counting number of reads mapped to each transcript

I/O       |   Format     | Description
--------- | ------------ | -----------------------
Input     |    BAM       |  Aligned raw reads
Output    |    TXT/TSV   |  Quantified expression value (TPM/FPKM)

***
## II. Preparation

We use [Bowtie](http://bowtie-bio.sourceforge.net/index.shtml) for alignment and [RSEM](https://deweylab.github.io/RSEM/) for gene quantification. We use reference trancripts and gene annotations from **Ensembl**. Follow the following steps for setting up:

* First, make sure C++, Perl, and R are installed. Also make sure to install `perl-doc` with command `apt-get install perl-doc`.
* Download the latest release of **Bowtie** from https://sourceforge.net/projects/bowtie-bio/files/bowtie/1.2.2/. Follow the instructions to install.
* Download the latest release of **RSEM** from https://github.com/deweylab/RSEM/archive/v1.3.1.tar.gz. Follow the instructions to install.
* Download and decompress the human genome file (FASTA) and annotation file (GTF):
`wget ftp://ftp.ensembl.org/pub/release-83/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz`
`wget ftp://ftp.ensembl.org/pub/release-83/gtf/homo_sapiens/Homo_sapiens.GRCh38.83.gtf.gz`
