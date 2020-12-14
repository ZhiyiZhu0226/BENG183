# RNA-seq data analysis pipeline

## Overview
This handout will cover detailed information about how to run each step of the RNA sequencing pipeline and how to interpret the input and output of each step. There are 8 steps in total:
1. Get raw reads data
2. Quality check of the raw reads
3. Clean raw reads
4. Map sequencing reads to genome
5. Alignment quality check
6. Quantify gene expression level
7. Normalize the read counts
8. Differential expression analysis


## Purpose of this pipeline

The ultimate goal is to measure the expression level of multiple genes and compare the expression levels across samples.

## 1. Get raw reads data
The input of this pipeline would be raw sequence data in the form of fastq files. You can obtain fastq files by sequencing your sample of interest, or download fastq files online using GEO/SRA.
Links to tutorials showing how to download fastq files using GEO/SRA:
[GEO Tutorial](https://www.imm.ox.ac.uk/files/ccb/downloading_fastq_geo)
[SRA Tutorial](https://www.youtube.com/watch?v=JvifigTF4yY)

The fastq file contains multiple sequences, and there will be 4 lines representing each sequence:
![fastqFile](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/fastq.PNG?token=ASCZPVEX7JSSG23S4KJZGKS723ODS)
1) Line 1 is the sequence id. Note that all sequence id begin with a “@”.
2) Line 2 is the actual sequence read composed of ATGCs.
3) Line 3 is a separator.
4) Line 4 is a sequence of phred quality scores, each corresponding to a base in line 2. For example, the green box shows the score for that base “T” is “G”. The scores are represented in ASCII values, and a greater score means less probability of error, i.e, higher accuracy and higher quality.
The following chars are used to represent phred quality scores:
 ``'!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~``
“!” represents the lowest quality (it has the smallest ASCII value) and “~” represents the highest quality.

**Tips:**
Counting the number of reads in a fastq file: Note that “@” can also be the first character of line 4. In other words, all sequence id starts with an “@”, but not all lines starting with “@” is a sequence id. Therefore, do not use grep -c '^@' filename.fq to count the number of reads. Use wc -l filename.fq instead. It will give the number of lines in the file, and you can get the number of reads by dividing that number by 4. The number of reads will also be included in the fastqc report.

## 2.Quality check of the raw reads
The first step is to check the quality of the raw reads using fastqc.
Get help:
`fastqc -h	//gives the description and usage of the fastqc package`

How to run fastqc:
`fastqc file1.fastq file2.fastq`

You can run fastqc on multiple files at a time. After running this command, you can see the progress of the program on the command prompt panel. The program will generate an html report.
![documents screenshot](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/html%20report.PNG?token=ASCZPVBCXDML7RDDEEK5ZIC723PPY)

![HTML report](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/fastqc-command%20line.PNG?token=ASCZPVAOVWLU2LTVBXTOJYK723PZY)
_(Screenshot of the HTML report; note that the number of reads is also included in the basic statistics)_

An alternative way of running fastqc is:
`fastqc`
This will open an interactive graphical application in which you can dyamically load fastq files and view their reports. If you are using Ubuntu to run fastqc, you may not be able to run fastqc interactively.
![application](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/fastqc-interactive.png?token=ASCZPVFP3PKW73AWI6BZFHS723QUM)
_([Screenshot](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
) of the interactive application)_

The fastqc report contains graphs and tables that shows the quality of the input fastq file. The index on the left allows you to jump to a specific graph by clicking on the corresponding link.
![index](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/fastqc%20index.PNG?token=ASCZPVF3HYYI3F2KADMILE2727RKA)

Each link corresponds to a test. The color of the bulletin points shows the quality of the corresponding module. Green represents pass, orange represents warning, and red represents failed. Although it seems like that more green bulletin points indicates a file has better quality, the thresholds used by the program may not suit your data, so your data could still have good quality even if it failed several tests. For example, the thresholds to determine the quality of a DNA file and a RNA file should be different, yet fastqc uses the same threshold and therefore produces inaccurate test results.
The following graphs are results from running fastqc on [female_midgut1_R1_raw.fastq.gz](http://sysbio.ucsd.edu/public/wenxingzhao/CourseFall2019/DS_raw/female_midgut1_R1_raw.fastq.gz), which is a RNAseq data of drosophila melanogaster.

#### How to interpret the test results
- Basic statics:
This table gives some simple composition statistics for the analyzed file. Note that for sequence length, it should provide the length of the shortest and longest sequence in the set. In the picture below, it’s only providing one value because all sequences in this file have the same length.
![Basic statics](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/basic%20statistics.PNG?token=ASCZPVFOR7LA73XBNN2STDK727R7Y)

- Per base sequence quality
The plot shows the Phred quality score of each base for all reads (Red: bad; Orange: reasonable; Green: good). The x axis is the position of each base, and the y axis is the Phred quality score. Shown is a good quality graph, whereas the score for most bases are in the green region.
![Per base sequence quality](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Per%20base%20sequence%20quality.png?token=ASCZPVDOOSZKDHSYJRZGHK2727SKY)

- Per tile sequence quality
This is a heatmap that gives the flow cell quality by individual tiles. The y-axis is a tile, the x axis is a base, and the color is the quality score. Having bright blue for most regions indicates good quality. Other colors indicate there are problems with specific tiles and sequence quality in those regions may be lower. The picture below shows there are potential problems for tile 2215-2305.
![Per tile sequence quality](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Per%20tile%20sequence%20quality.png?token=ASCZPVGPQLEPHVQ6AZH5HRK727SVA)

- Per base sequence content
This graph shows the percentage of each base (ATCG) at each position of all reads. When analyzing RNA, this test will likely fail or produce warning because the primers used during PCR have arbitrarily chosen sequences. This test will still fail even if we cut the reads, yet in most cases, this won't have much of a negative effect on the following steps in the pipeline.
![Per base sequence content](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Per%20base%20sequence%20content.png?token=ASCZPVGEU7HLT5FBTYTKHVC727TJI)

- Per sequence quality scores
The y-axis is the number of reads, and the x-axis is the mean Phred score of a read. The graph below shows that most reads have high average Phred scores, and thus most reads have high quality.
![Per sequence quality scores](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Per%20sequence%20quality%20scores.png?token=ASCZPVHLY63JHSROQJWIESS727TTW)

- Per sequence GC content
Y-axis stands for the number of reads with a GC presentation of x, and x-axis stands for the GC content. Theoretically the distribution should be normal, and any other distribution may indicate contamination or adapter sequences. (This graph shows a bad GC content)
![Per sequence GC content](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Per%20sequence%20GC%20content.png?token=ASCZPVEJ6UJWISWFYXSOLT2727T26)

- Per base N content
This graph shows the percentage of undefined base at certain positions among all reads. The following graph shows a good result that has a flat line with y=0. Consider resequencing if y is too high at any position.
![Per base N content](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Per%20base%20N%20content.png?token=ASCZPVHPECNM6NAOF3O6O3K727UBO)

- Sequence Length Distribution
X-axis is the length of a read, and y-axis is the number of reads that has length of x. Since we haven’t trimmed the raw reads yet, all the reads should have a length of 125.
![Sequence Length Distribution](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/Sequence%20Length%20Distribution.png?token=ASCZPVF4L7ANDIWOBSILMQC727UHA)

- Sequence Duplication Levels
The x-axis shows a sequence is found in the file for how many times, and the y-axis shows the percent of reads with that sequence. High duplication levels could come from adapters, biased enrichment during PCR, or transposons. However, it’s common to have high duplication levels when sequencing RNA since some gene fragments have higher expressions and thus have more duplications. Due to the threshold used by the program, this test will likely fail even the duplication level is expected in this case.
![Sequence Duplication Levels](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/sequence%20duplication%20levels.png?token=ASCZPVGV3CGUWFSWQIG5H42727UM2)

- Overrepresented sequences
This table lists all of the reads that make up more than 0.1% of the total. Overrepresented sequences could result from contamination, unremoved adapters, or genes that are highly significant.
![Overrepresented sequences](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/overrepresented%20sequences.PNG?token=ASCZPVFYXR6JQW5FTCIFCO2727UVI)

- Adapter content
This graph searches for adapters and shows the location of the adapters. Any line that’s not flat with y=0 indicates there are unremoved adapters.
![Adapter content](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/adapter%20content.png?token=ASCZPVHRE6HWXVMLKMQHQSS727U3Q)

Based on the above interpretation, although the file failed several tests, it still has
relatively good quality. We don’t need to resequence in this case, but we need to clean the data before mapping it to the genome.

## 3.Clean raw reads

In this step, we will be using fastp to clean the raw data. Specifically, fastp will remove low quality reads and adaptor sequences. Doing so can reduce the chance for a read to be mapped to a wrong position or multiple positions, and thus increase the mapping quality.  [Here](https://github.com/OpenGene/fastp) is a detailed guide about using fastp.

Before running fastp, check if your data is single-end sequenced or paired-end sequenced.
For single sequenced data:
`fastp -i sample_raw.fastq -o sample_clean.fastq -h report_file_name.html //the h flag is optional`

For paired-end sequenced data:
`fastp -i sample_R1_Raw.fastq -I sample_R2_Raw.fastq -o sample_R1_clean.fastq -O sample_R2_clean.fastq -h sample_fastp.html`

After running fastp, it will print a summary on the command prompt, and it will also output the cleaned fastq files and a fastp report to the current folder. Note that for paired sequenced data, it will output 2 cleans files and 1 html report.
![fastp termial output](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/fastp%20temrial%20output.PNG?token=ASCZPVBKWNH3E4XWJI4NXX2727VPI)

![fastp report](https://raw.githubusercontent.com/ZhiyiZhu0226/BENG183/main/fastp%20report.PNG?token=ASCZPVFQKG5WDBGA7SBBGI2727VR2)

You can run fastqc on the cleaned file to see if the quality has improved. Keep in mind that running fastp won’t deal with all the abnormalities and some abnormalities are just meant to be there. You should see the cleaned file passing the adapter content test, but you will also see a warning or fail for the Sequence Length Distribution test. This is because bases are trimmed from the reads and the length of each read will vary depending on how many bases are cut off. Again, passing a test doesn’t necessarily mean good quality.
