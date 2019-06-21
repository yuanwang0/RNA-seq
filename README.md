# RNA-seq

***
## I. Overview

Here we demonstrate how to convert raw `.fastq` files into quantified expression files for RNA-seq data.

We use [STAR](https://github.com/alexdobin/STAR) for alignment and [RSEM](https://deweylab.github.io/RSEM/) for gene quantification. The entire RSEM pipeline can be broken down into two major parts:
#### Step 1: Build reference transcripts from a reference genome and gene annotations

I/O       |   Format   | Description
--------- | ---------- | ---------------------------------
Input     |   FASTA    |  Reference genome
Input     |    GTF     |  Gene Annotations
Output    |   FASTA    |  Extracted reference transcripts

#### Step 2: Quantify and normalize expression levels by counting number of reads mapped to each transcript

I/O       |   Format     |   Description
--------- | ------------ | --------------------------------------------------
Input     |   FASTQ      |  Unaligned raw reads
Output    |    BAM       |  Aligned reads (either transcript- or gene-level)
Output    |   RESULTS    |  Tab-delimited quantified expression (either transcript- or gene-level)

***
## II. Preparation

We use reference trancripts and gene annotations from [Ensembl](http://useast.ensembl.org/info/data/ftp/index.html). Here is an example of the directory in tree setup:
```
RNA-seq
      |-- sw
            |-- RSEM-1.3.1
            |-- STAR-2.7.1
      |-- data
            |-- controls
            |-- CHAMP1_patients
      |-- ref
      |-- out
```


Follow the following steps:

* First, make sure C++, Perl, and R are installed. Also make sure to install `perl-doc` with command `apt-get install perl-doc`.
* Download the latest release of **STAR** from https://github.com/alexdobin/STAR/archive/2.7.1a.tar.gz to `sw/`. Follow the instructions to install.
* Download the latest release of **RSEM** from https://github.com/deweylab/RSEM/archive/v1.3.1.tar.gz to `sw/`. Follow the instructions to install.
* Download the **Ensembl** human genome file (FASTA) and annotation file (GTF) to `ref/`:

```
ftp://ftp.ensembl.org/pub/release-83/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
ftp://ftp.ensembl.org/pub/release-83/gtf/homo_sapiens/Homo_sapiens.GRCh38.83.gtf.gz
```

Then use the following commands to decompress the files:

```
gunzip /ref/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
gunzip /ref/Homo_sapiens.GRCh38.83.gtf.gz
```

***
## III. RSEM - step 1: build reference transcripts

```
#!/bin/bash

RSEM_dir="/sw/RSEM-1.3.1"
star_dir="/sw/STAR-2.7.1"
ref_dir="/ref"

${RSEM_dir}/rsem-prepare-reference --gtf ${ref_dir}/Homo_sapiens.GRCh38.83.gtf \
	                             --star \
		                       --star-path ${star_dir} \
		                       ${ref_dir}/Homo_sapiens.GRCh38.dna.primary_assembly.fa \
		                       ${ref_dir}/human_ensembl

```

#### Parameters explained
* `--gtf`: Specifies that `${ref_dir}/Homo_sapiens.GRCh38.dna.primary_assembly.fa` is a genome sequence, hence will extract reference transcripts using gene annotation file `${ref_dir}/Homo_sapiens.GRCh38.83.gtf`.
* `--star`: Build STAR genome indexes.
* `--star-path`: Path to STAR's executables.
* `${ref_dir}/Homo_sapiens.GRCh38.dna.primary_assembly.fa`: Argument 1 *reference_fasta_file(s)*
* `${ref_dir}/human_ensembl`: Argument 2 *reference_prefix_name*: 
