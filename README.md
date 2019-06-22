# RNA-seq

***
## I. Overview

Here we demonstrate how to convert raw `.fastq` files into quantified expression files for RNA-seq data.

We use [STAR](https://github.com/alexdobin/STAR) for alignment and [RSEM](https://deweylab.github.io/RSEM/) for gene quantification. The entire RSEM pipeline can be broken down into two major steps:

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
* Download the latest release of **STAR** from https://github.com/alexdobin/STAR/archive/2.7.1a.tar.gz to `sw/`. Follow the instructions to install. Since STAR is the only aligner built in RSEM that supports gunzipped FASTQ files as input, we prefer it for the sake of memory efficiency.
* Download the latest release of **RSEM** from https://github.com/deweylab/RSEM/archive/v1.3.1.tar.gz to `sw/`. Follow the instructions to install. For a complete introduction to RSEM please see [here](https://deweylab.github.io/RSEM/); otherwise, continue reading this README for an introduction to the part of RSEM used in this analysis.
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
* `${ref_dir}/Homo_sapiens.GRCh38.dna.primary_assembly.fa`: *Argument 1*, reference_fasta_file(s) in FASTA format.
* `${ref_dir}/human_ensembl`: *Argument 2*, reference_prefix_name.
* `--gtf <file>`: Specify that *Argument 1* is a genome sequence, hence will extract reference transcripts using gene annotation file `<file>`.
* `--star`: Build STAR genome indexes.
* `--star-path`: Path to STAR's executables.

For a more detailed documentation of all parameters, please consult `rsem-prepare-reference --help`.

***
## IV. RSEM - step 2: calculte expression values

Here we use the control sample `HH07` as an example. This data is **paired-end** and **non-strand specific**. You may read more about **paired-end sequencing** [here]((https://www.illumina.com/science/technology/next-generation-sequencing/paired-end-vs-single-read-sequencing.html)).

*See here for some ways of guessing the strandedness of an RNA-seq library.*


```
#!/bin/bash

RSEM_dir="/sw/RSEM-1.3.1"
star_dir="/sw/STAR-2.7.1"
data_dir="/data/controls"
ref_dir="/ref"
out_dir="/out

${RSEM_dir}/rsem-calculate-expression -p 1 \
                                      --paired-end \
				      --star \
				      --star-path ${star_dir} \
				      --estimate-rspd \
				      --append-names \
				      --output-genome-bam \
				      --star-gzipped-read-file \
				      ${data_dir}/HH07_1.fq.gz \
				      ${data_dir}/HH07_2.fq.gz \
				      ${ref_dir}/human_ensembl \
				      ${out_dir}/control.HH07
```

#### Parameters explained
* `${data_dir}/HH07_1.fq.gz`: *Argument 1*, upstream_read_file(s) for paired-end data in FASTQ format.
* `${data_dir}/HH07_2.fq.gz`: *Argument 2*, downstream_read_file(s) for paired-end data in FASTQ format.
* `${ref_dir}/human_ensembl`: *Argument 3*, same as the 'reference_prefix_name' in `rsem-prepare-reference`.
* `${out_dir}/control.HH07`: *Argument 4*, output_prefix_name.
* `-p`: Number of threads to use; corresponds to the `--runThreadN` option in STAR.
* `paired-end`: Specify that the input reads are paired-end.
* `--star`: Use STAR to align reads. Alignment parameters are from ENCODE3's STAR-RSEM pipeline.
* `--estimate-rspd`: Estimate the read start position distribution (RSPD) from data.
* `--append-names`: Append gene names to the end of 'gene_id' (which is Ensembl ID) in the the quantification results files, `<output_prefix>.isoforms.results` and `<output_prefix>.genes.results`.
* `--output-genome-bam`: Output a BAM file with alignments mapped to genomic coordinates and annotated with their posterior probabilities. STAR's BAM output is unsorted.
* `--star-gzipped-read-file`: Input files are gzipped.

For a more detailed documentation of all parameters, please consult `rsem-calculate-expression --help`.

***
## V. Build gene expression profile for URSA(HD) input

Let's first take a look at first two rows of the output file, `control.HH07.genes.results`, from RSEM step 2 `rsem-calculate-expression`.

```
gene_id	transcript_id(s)	length	effective_length	expected_count	TPM	FPKM
ENSG00000000003_TSPAN6	ENST00000373020_TSPAN6-001,ENST00000494424_TSPAN6-002,ENST00000496771_TSPAN6-003,ENST00000612152_TSPAN6-201,ENST00000614008_TSPAN6-202	2180.18	1925.63	628.00	13.05	9.65
```

The goal is to create a file whose first column contains gene names (e.g BRCA1) and second column contains quantified expression values by
1. Extracting gene names from the 'gene_id' column. For example, we want `TSPAN6` instead of `ENSG00000000003_TSPAN6`;
2. Selecting of column of the appropriate expression unit (TPM or FPKM) as our gene expression value. Here we use TPM as an example;
3. Removing the header row and save the output to a `.pcl` file. Note that the `.pcl` suffix does not really matter; however, we still save the file to `.pcl` to keep things consistent with sample URSA(HD) input files.


The following command creates such a file:
```
#!/bin/bash

RSEM_results=control.HH07.genes.results

cut -f1,6 ${RSEM_results} | sed 's/^.*_//' | tail -n +2 > ${RSEM_results}.TPM.pcl
```

We have thus created a file `control.HH07.genes.results.TPM.pcl` that can be used as input for URSA(HD)!

***
## VI. References

B. Li, C.N. Dewey. RSEM: accurate transcript quantification from RNA-Seq data with or without a reference genome. BMC Bioinform, 12 (1) (Dec. 2011), p. 323.

A. Dobin, T.R. Gingeras. Mapping RNA-seq reads with STAR. Curr Protoc Bioinformatics, 51 (Sep. 2015), pp. 11.14.1-19

Lee YS, Krishnan A, Oughtred R, Rust J, Chang CS, Ryu J, Kristensen VN, Dolinski K, Theesfeld CL, Troyanskaya OG.
A Computational Framework for Genome-wide Characterization of the Human Disease Landscape. Cell Systems, 8 (2) (Feb. 2019), p. 152-162.e6.
