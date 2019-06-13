# RNA-seq

Here we demonstrate how to convert raw `.fastq` files into quantified expression files for RNA-seq data.

The entire pipeline can be broken down into two major parts:
### Step 1: Align reads into a reference genome

I/O       |   Format   | Description
--------- | ---------- | -----------------------
Input     |   FASTQ    |  Unaligned raw reads
Output    |    BAM     |  Aligned reads

### Step 2: Quantify and normalize expression levels by counting number of reads mapped to each transcript

I/O       |   Format     | Description
--------- | ------------ | -----------------------
Input     |    BAM       |  Aligned raw reads
Output    |    TXT/TSV   |  Quantified expression value (TPM/FPKM)
