# Sequencing data formats and visualization 
In this computer lab we will have a look at the most common file formats used for next-generation sequencing data. 

## Overview

1. Examining and understanding FastQ files
2. Investigating mapped information in BAM files
3. Evaluating variants using VCF files


## Prerequisites
For this, you should have IGV installed on your computer. If you haven't done so yet, please download it [here](https://igv.org/). There is also a web application in case you can't get it set up, though functionality is decreased. 

We will be performing the analysis on uppmax and on your computer locally. To get set up you need to load the following modules on uppmax:

```text
module load IGV
module load SAMtools/1.22
module load BCFtools/1.22
```

## Files
Copy the files you need to your home directory: 

```text
cd ~
mkdir lab1
rsync /proj/uppmax2024-2-1/nobackup/lab_files/lab1_data_format/* ~/lab1/.
```

The samples you will be working with in this lab are both from a germline sample (healthy tissue) as well as a somatic sample (blood cancer). 

## Resources
* [Genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/)
* [IGV desktop Documentation](https://igv.org/doc/desktop/)
* [ASCII table](https://en.wikipedia.org/wiki/ASCII#Printable_characters)
* [FastQ Format](https://en.wikipedia.org/wiki/FASTQ_format)
* [Samtools](https://www.htslib.org/doc/samtools.html)
* [bitwise flags](https://broadinstitute.github.io/picard/explain-flags.html)
* [bcftools](https://samtools.github.io/bcftools/bcftools.html)


## 1. FASTQ files
To store the countless reads generated in short read sequencing along with additional information, the FASTQ file format was developed as a progression of the FASTA file format.  

FASTQ files often contain multiple reads where each read is described using four rows:
 
1. A sequence identifier, row starts with @ 

2. the raw read 

3. \+

4. Quality scores as [PHRED score](https://en.wikipedia.org/wiki/FASTQ_format#Encoding) 

Below you can see an example of fastq file showing three reads. Have a look at the example to see if you can identify the rows: 


```text
@2fa9ee19-5c51-4281-abdd-eac8663f9b49 runid=f53ee40429765e7817081d4bcdee6c1199c2f91d sampleid=18S_amplicon read=109831 ch=33 start_time=2019-09-12T12:05:03Z
CGGTAGCCAGCTGCGTTCAGTATGGAAGATTTGATTTGTTTAGCGATCGCCATACTACCGTGACAAGAAAGTTGTCAGTCTTTGTGACTTGCCTGTCGCTCTATCTTCCAGACTCCTTGGTCCGTGTTCAATCCCGGTAGTAGCGACGGGCGGTGTATGTATTATCAGCGCAACAGAAACAAAGACACC
+
+&&-&%$%%$$$#)33&0$&%$''*''%$#%$%#+-5/---*&&%$%&())(&$#&,'))5769*+..*&(+28./#&1228956:7674';:;80.8>;91;>?B=%.**==?(/'($$$$*'&'**%&/));807;3A=;88>=?9498++0%"%%%%'#&5/($0.$2%&0'))*'%**&)(.%&&
@1f9ca490-2f25-484a-8972-d60922b41f6f runid=f53ee40429765e7817081d4bcdee6c1199c2f91d sampleid=18S_amplicon read=106343 ch=28 start_time=2019-09-12T12:05:07Z
GATGCATACTTCGTTCGATTTCGTTTCAACTGGACAACCTACCGTGACAAAGAAAGTTGTCGATGCTTTGTGACTTGCTGTCCTCTATCTTCAGACTCCTTGGTCCATTTCAAGACCAAACAATCAGTAGTAGCGACGGGCGGTGTGGCAATATCGCTTTCAACGAAACACAAAGAAT
+
&%&%''&'+,005<./'%*-)%(#$'$#$%&&'('$$..74483=.0412$*/,)9/194/+('%%(+1+'')+,-&,>;%%.*@@D>>?)3%%296070717%%16;<'<236800%(,734(0$7769879@;?8)09:+/4'1+**7<<4.4,%%(.)##%&'(&&%*++'&#%$
@06936a64-6c08-40e9-8a10-0fbc74812c89 runid=f53ee40429765e7817081d4bcdee6c1199c2f91d sampleid=18S_amplicon read=83531 ch=23 start_time=2019-09-12T12:03:50Z
GTTTTGTCGCTGCGTTCAGTTTATGGGTGCGGGTGTTATGATGCTTCGCTTTACGTGACAAGAAAGTTAGTAGATTGTCTTTATGTTTCTGTGGTGCTGATATTGCCACACCGCCCGATAGCTCTACCGATTGAAACACGGACCAAGGAATCGGAAATGTAGGCGAGCAGGCCGTCCTGAACACCCATTAACTTTCTTGTC
+
$&'((&%$$$.$2/=-*#'.2'&&##$$#$#&&(&+-%'(%&#"###""$$%#)%,+)+&'(,&%*((%%&%$%'+),,+,,&%$')1+*$.&+6*+(*%(&'*(''&%*+,*)('%#$$$%,$&&'&)))12)*&/*,364$%$%))$'')#%%&%$#$%$$#('$(%$%$%%$$*$&$%)''%%$$&'&$)+2++,)&%
```

If sequencing was performed on paired reads, you will always have at least two .fastq files per sample, marked with R1 and R2, for read1 and read2. They are sorted so that the first read in R1 is the mate of the first read in R2. They should, therefore, always have the same number of rows in each file.


### Assignment 1 

In `~/lab1/` you will find an example of a FASTQ file generated with a NovaSeq sequencing machine: `q1.fq`

!!! question "Question 1"

	:question: 
	Have a look at the [sequence identifier](https://en.wikipedia.org/wiki/FASTQ_format#Illumina_sequence_identifiers) of the FASTQ file. The format used in this file follows the changes after Casava 1.8. 

	* Is this FASTQ file from paired end sequencing? (try to infer this from the sequence identifier, maybe the second file wasn't copied over)
	* Which read pair are we looking at? 
	* What is the index of this sample? 


:question:
Have a look at the the characters in the quality row, what do you think is happening here? 
   
!!! question "Question 2"

	:question: 
	What Phred score value do the symbols you see correspond to? Here is a link to an [ASCII table](https://en.wikipedia.org/wiki/ASCII#Printable_characters). Remember, we are counting from 33 in the decimal system (example: ! is 33 in the decimal system, Therefore the Phred score value is 33-33=0). 

## 2. BAM
Aligned reads are stored in .sam/.bam/.cram files. While the .sam file is a regular text file, .bam is packaged as a binary file and therefore needs less disk space. A further compression is the .cram file which can be lossless or lossy. 
The information in these files are structured tab-delimited and contain a header that is prior to the read data. The header rows start with an @. After the header follow the aligned reads, with one row per read. 


When working with the binary .bam and .cram files programs often need an index file to make computation quicker. Index files function like a table of content in a book, pointing the reader to content without the need to read the book until you find the requested information. 


### Assignment 2 

In your terminal look and compare the .bam files in the directory. 
Remember, .bam files are binary files, so using commands like `cat` or `less` will only results in gibberish on your screen. 
Thankfully, there are programs that help you handling binary files. To explore .bam or .cram files you can use the program **samtools**. 
[Samtools](https://www.htslib.org/doc/samtools.html) has a range of utilities to process sam/bam/cram files. These sub-commmands can be used like `samtools subcommand <optional flags>`. Each subcommand has it's own documentation page. Have a look at the documentation for [samtools view](https://www.htslib.org/doc/samtools-view.html), which is what we'll be using to investigate the .bam files. These files are really big so make sure to pipe into `less` or you will print the entire file in your terminal. 



:question:
Try using samtools view on sample_C.bam. What do you see? Can you identify the columns we discussed in the lecture part?


!!! question "Question 3"

	:question:
	Using samtools view without any flags will only show us the reads. Even the column header is missing. However, .bam files store this meta-information in the header of the .bam file. Look at the documentation to find the flag to display the header. 

	Notice that there are tags in the header. You can check what these mean in the [samtools manual](https://samtools.github.io/hts-specs/SAMv1.pdf). In the file sample_C.bam, which programs have been used to create the bam file? 


!!! question "Question 4"
	:question: 
	Figure out which flag to use in samtools view to count the number of reads in the file. What is the number of reads in sample_C.bam?


:question: 
Have a look at the FLAG field and pick out a number. Check the bitwise flags on [wikipedia](https://en.wikipedia.org/wiki/SAM_(file_format)#Bitwise_flags) to calculate what this number is encoding. 

??? info 
	You can check if your calculations were correct and to have a look at some other numbers using this [website](https://broadinstitute.github.io/picard/explain-flags.html)


### IGV - Integrative Genomics Viewer  

To help visualize mapped reads, tools such as IGV have been developed. We can use IGV to investigate variants in more detail along with their position on the reads. 



### Assignment 3 

:question:
In your next assignment we want to look at the reads with IGV. IGV needs index files to the bam files, however, this is missing from **sample_A.bam**. Indexing will only work on sorted bam files, and it is good practice to perform sorting before indexing. For this, you can use further samtools subcommands, aptly named `sort` and `index`. Find the [documentation](https://www.htslib.org/doc/samtools.html) for these subcommands and create an index file. You should end up with files called 'sample_A.sorted.bam' and 'sample_A.sorted.bam.bai'.
*sample_B.bam and sample_C.bam are already sorted and have index files, no need to create new files and indeces here.*

You should then download the bam files and bam index files to your computer (sample_A.sorted and sample_C). **sample_B.bam** and it's index should have already been downloaded prior to the start of the lab as the file size is massive. You can download with scp. In a terminal window navigate to a local directory you want to download the data to and run: 

```text
scp "<USER>@pelle.uppmax.uu.se:~/lab1/sample_{A,C}*.bam*"  . 
```
exchange &lt;USER> with your uppmax user name. The command will prompt you to type in your uppmax password and then will transfer the data to the directory you are in. 


Now open IGV and select reference hg19. Load the .bam files of sample_A.sorted, sample_B and sample_C into IGV. Select the .bam files and make sure that the .bai files are in the same directory. You can either use IGV on the [compute cluster](https://docs.uppmax.uu.se/software/igv/), or you can download the files and start IGV locally on your computer (**recommended**). You can use the command `igvtools_gui` on uppmax to open IGV but this is not recommended as the graphic forwarding is not that great (and need login with ssh -Y). If you have issues opening IGV locally there is also an [IGV web app](https://igv.org/app/).

??? info
	Once you input data, you can click on a read and get more information about the read. If you right click on a read, you have the option of visualizing different features such as mate pairs, or mapping quality.  It is often helpful to right click on a read and sort the reads by base, the position you are interested in should be in the center. If you do not have a center line click on 'View' on the bar on top the 'Preferences...' go to the tab 'Alignment' and make sure that 'show center line' is turned on. 


	If you click on the coverage plot part at the top you get more information on read depth of the base and the allele frequency of the variants found in this position. You can also see which strand they were found on (+ for forward and - for reverse). 


!!! question "Question 5"

	:question:
	Navigate to the gene ABL1. Look at the reference at the bottom to find where the exons are and where targeted regions might be sequenced. Zoom in closer and navigate to exonic sites. What is the difference between the different .bam files? See how the reads are mapped and also look at the coverage plots at the top of each .bam file. The samples we are looking at where sequenced with three different strategies, namely whole genome, amplicon and capture. Can you find out which sample was sequenced with which strategy?  
         

!!! question "Question 6"

	:question: 
	Navigate to position 133748283 on this chromosome. There is a variant here in the sample sequenced with the capture method. You can get a read count and allele frequency when clicking at the position in the coverage plot.  
        What is the allele frequency and read depth at this position for the capture sequence data? What about the amplicon sequence data? And what about the WGS sample? 


	!!! note 
		Notice how the WGS sample looks very messy, as if most bases carry a variant? Do you have an idea why? Remove the sample from IGV, we will look at it later. You can do this by right clicking on the sample name on the left and then *remove track*  at the very bottom.


!!! question "Question 7"
	:question:
	Navigate to chr9:5070022.  Which gene are we looking at? What variant do you observe at this position in the capture and amplicon sample. How long is the variant? What is the affected sequence? 
     
!!! question "Question 8"
	:question:
	Navigate to chr9:5073770. What type of variant do you find here? What is the read depth, and allele frequencies in the different .bam files (sample A and C)? 


!!! question "Question 9"
	:question:
	Navigate to chr5:170837544. Which gene are we looking at? What variant do you observe at this position  in the capture and amplicon sample. How long is the variant? What is the affected sequence?

 
:question:
Navigate to chr5:170846329-170847798. Look at the sample sequenced with the capture method. Make sure to scroll all the way up so you can see the coverage plot. What is the read depth here? How does this compare to the read depth in other areas we looked at. Check the reference, are you looking at a gene, and if yes, what part of the gene? What do you think is happening here?   


:question:
The WGS sample was mapped to hg38. Lets switch the reference accordingly and open the sample again. Navigate to chromosome 9. Do you notice a difference to how it looked like before? Try to find some SNVs, what are their allele frequencies? Mark down a few that you can find along with how many reads support them and how many strands are forward and reverse. Which do you think are real variants and which do you think are errors/artefacts?

??? note "Artefacts"
	Artefacts can be introduced at different steps during extraction, amplification, sequencing or mapping. To determine whether we think something is an artefact there are a few things we can look for in IGV. 

	* Are there several reads supporting the variant call? 
	* Can the variant be found only at the ends of reads? (Variants in middle of reads are more reliable)
	* Do both forward and reverse strands support the variant in a somewhat equal manner? 
	* Is there a lot of noise surrounding the variant? 

	Additionally, artefacts often occur in the same spot of the genome when following the same protocols. So once we collect enough data, we can determine whether variants occur too often across our cohort in low allele depth to be considered a real variant. 


??? "germline vs somatic"
	From germline samples you expect less variants than in somatic (tumor) samples. Also generally you expect the allele frequencies in diploid germline samples to be around 0%, 50% or 100% as the variation is inherited. Variation in somatic samples can be anything in the range of 0-100% due to rising mutations and subclones. 


!!! question "Question 10"
	:question: 
	Switch the reference genome to hg19 again and look at sample_C. Look again at the variant in detail chr9:133748283. Is this an artefact? Why - why not? 



## VCF

VCF stands for **v**ariant **c**all **f**ormat. A vcf file is used to store variations of the genome encountered in a sample or data set. Each row in a vcf file describes a variant of a specific genomic position in your sample(s). You can either save all sites of the genome in a vcf-file, often referred to as a genome-vcf or it can contain just sites that differ from  the reference. You can save any type of variant in a vcf file, such as single nucleotide variations, insertions and deletions, or translocations. 

As .bam files, .vcf files also contain a header that describes the information found in the file. In the header you can also find out which vcf version is used to encode information in this file. You can always check the specific documentation for the vcf version if you do not know how to interpret the data. PDFs of the documentation of the most recent versions of vcf can be found [here](https://github.com/samtools/hts-specs). 

One method to filter and edit .vcf files is [bcftools](https://samtools.github.io/bcftools/bcftools.html), which functions similarly to samtools. 

### Assignment 4

In this assignment we will look at variants in the vcf file. Some of them we have already looked at in IGV, and you can always compare how the variants look like in the .vcf and in the .bam file. We will do this on UPPMAX again, so open up the terminal and navigate to `~/lab1`


!!! question "Question 11"

	:question:
	Look at the files sample_C.vcf.gz and sample_B.vcf.gz. Are they genome-vcf-files? Explain! 


:question: 
Search for the variant in chromosome 9 position 133748283 in sample_C.vcf.gz. It's a position in the ABL gene we have looked at before (Question 10). What is the variant frequency and read depth in the vcf file? How many reads have found either allele? Check the header to understand which tags you are looking for. How do the values compare to the values observed in IGV?

!!! question "Question 12"
	:question:
	Search for the variant chr9	37025419 in sample_C.vcf.gz. Check the FILTER column and look at the header in sample_C.vcf.gz, what does the flag in the filter field indicate? Look at this position in IGV, how many reads are found on the forward, how many on the reverse strand? How is it for the reference allele? Why do you think this is?    

!!! question "Question 13"
	:question:
	Using bcftools, figure out how many variants were found on chromosome 9  in sample_C.vcf.gz that passed all filters (FILTER="PASS") in sample_C.vcf.gz. Make sure you are not including the header when counting 


If you are done with all the exercises here, there is **one more question for you on the quiz on studium! **

 





