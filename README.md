# RNA-seq

***
## I. Overview

Here we demonstrate how to convert raw `.fastq` files into quantified expression files for RNA-seq data.

We use [STAR](https://github.com/alexdobin/STAR) for alignment and [RSEM](https://deweylab.github.io/RSEM/) for gene quantification. The entire RSEM pipeline can be broken down into two major parts:
#### Step 1: Generate a set of reference transcripts from a reference genome and gene annotations

I/O       |   Format   | Description
--------- | ---------- | -----------------------------
Input     |   FASTA    |  Reference genome
Input     |    GTF     |  Gene Annotations
Output    |   FASTA    |  Extracted transcripts
Output    |            |  STAR indices

#### Step 2: Quantify and normalize expression levels by counting number of reads mapped to each transcript

I/O       |   Format     | Description
--------- | ------------ | -----------------------
Input     |   FASTQ      |  Unaligned raw reads
Input     |    BAM       |  Aligned raw reads
Output    |    TXT/TSV   |  Quantified expression value (TPM/FPKM)

***
## II. Preparation

We use reference trancripts and gene annotations from **Ensembl**. Here is an example of the directory in tree setup:
```
RNA-seq
      |-- sw
      |-- data
            |-- control
      |-- ref
      |-- out
```


Follow the following steps:

* First, make sure C++, Perl, and R are installed. Also make sure to install `perl-doc` with command `apt-get install perl-doc`.
* Download the latest release of **STAR** from https://github.com/alexdobin/STAR/archive/2.7.1a.tar.gz to `sw`. Follow the instructions to install.
* Download the latest release of **RSEM** from https://github.com/deweylab/RSEM/archive/v1.3.1.tar.gz to `sw`. Follow the instructions to install.
* Download and decompress the human genome file (FASTA) and annotation file (GTF) to `ref`:

`ftp://ftp.ensembl.org/pub/release-83/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz`

`ftp://ftp.ensembl.org/pub/release-83/gtf/homo_sapiens/Homo_sapiens.GRCh38.83.gtf.gz`

***
## III. RSEM - step 1: build reference transcripts



