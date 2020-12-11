# RNA-seq data analysis pipeline

## Overview
This handout will cover detailed information about how to run each step of the RNA sequencing pipeline and how to interpret the input and output of each step.

## 1. Purpose of this pipeline

The ultimate goal is to measure the expression level of multiple genes and compare the expression levels across samples.

## 2. Get raw reads data
The input of this pipeline would be raw sequence data in the form of fastq files. You can obtain fastq files by sequencing your sample of interest, or download fastq files online using GEO/SRA.
Links to tutorials showing how to download fastq files using GEO/SRA:
[GEO Tutorial](https://www.imm.ox.ac.uk/files/ccb/downloading_fastq_geo)
[SRA Tutorial](https://www.youtube.com/watch?v=JvifigTF4yY)


The fastq file contains multiple sequences, and there will be 4 lines representing each sequence:
![fastqFile](https://drive.google.com/file/d/1C7cP-VXmW_foFNACaizyG0Ht8Yk3BR6e/view?usp=sharing)

1) Line 1 is the sequence id. Note that all sequence id begin with a “@”.
2) Line 2 is the actual sequence read composed of ATGCs.
3) Line 3 is a separator.
4) Line 4 is a sequence of phred quality scores, each corresponding to a base in line 2. For example, the green box shows the score for that base “T” is “G”. The scores are represented in ASCII values, and a greater score means less probability of error, i.e, higher accuracy and higher quality. The following chars are used to represent phred quality scores:
 ``'!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~``
“!” represents the lowest quality (it has the smallest ASCII value) and “~” represents the highest quality.
