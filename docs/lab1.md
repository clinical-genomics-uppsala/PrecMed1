# Sequencing data formats and visualization 
In this computer lab we will talk about and have a look at the most common file formats used for next-generation sequencing data. 

## FASTQ
To store the countless reads generated in short read sequencing along with additional information, the FASTQ file fomrat was developped as a progression of the FASTA file format.  

FASTQ files often contain multiple reads that are described in four rows:
 
1. A sequence identifier, row starts with @ 

2. the raw read 

3. + 

4. Quality scores as [PHRED score](https://en.wikipedia.org/wiki/FASTQ_format#Encoding) 

Example of fastq file: 
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

If sequencing was performed on paired reads, you will always have at least two .fastq files per sample, marked with R1 and R2, for read1 and read2. These will be sorted so that the first read in R1 is the mate of the first read in R2. 


### Assignment 1 
!!! question 
	In `PATH` you will find an example of a fastq file generated with a NovaSeq sequencing machine: `q1.fq`

	:question:
	Have a look at the [sequence identifier](https://en.wikipedia.org/wiki/FASTQ_format#Illumina_sequence_identifiers). Is this fastq file from paired end sequencing? Which read pair are we looking at? What is the index of this sample? 

	:question:
	Have a look at the quality scores, what do you think is happening here? 
     

	:question:
	What Phred score value do the symbols you see correspond to? Here is a link to an [ASCII table](https://en.wikipedia.org/wiki/ASCII#Printable_characters). Remember, we are counting from 33 in the decimal system. 

## BAM
Output from mapping is stored in .sam/.bam/.cram files.
These files are tab-delimited and container a header, where header rows start with an @, and the alignment data, with one row per aligned sequence. 


### Assignment 2 

In your terminal look and compare the bam files in the directory. 
Remember, bam files are binary files, so using commands like `cat` or `less` will only results in gibberish on your screen. 
Thankfully, there are programs that help you handling binary files. To explore bam files one of these programs is **samtools**. 
[Samtools](https://www.htslib.org/doc/samtools.html) has a range of utilities to process sam/bam/cram files. These sub-commmands can be used like `samtools subcommand <optional flags>`. Each subcommand has it's own documentation page. Have a look at the documentation for [samtools view](https://www.htslib.org/doc/samtools-view.html), which is what we'll be using to investigate the bam files. These files are big so make sure to pipe into `less`. 

!!! question
	
	:question:
	Try using samtools view on sample_C.bam. What do you see? Can you identify the columns we discussed in the lecture part?

	:question:
	Using samtools view without any flags will only show us the reads. Even the column header is missing. However, bam files store this meta-information in the header of the bam file. Look at the documentation to find the flag to display the header. 
	Notice that there are tags in the header. You can check what these mean in the [samtools manual](https://samtools.github.io/hts-specs/SAMv1.pdf). In the file sample_C.bam, which programs have been used to create the bam file? 

	:question: 
	Figure out which flag to use in samtools view to count the number of reads in the file. What is the number of reads in sample_C.bam?

	:question:
	In your next assignment we want to look at the reads with IGV. IGV needs index files to the bam files, however, this is missing from sample_A.bam. Indexing will only work on sorted bam files, and it is good practice to perform sorting before indexing. For this, you can use further samtools subcommands, aptly named `sort` and `index`. Find the [documentation](https://www.htslib.org/doc/samtools.html) for these subcommands and create an index file. You should end up with files called 'sample_A.sorted.bam' and 'sample_A.sorted.bam.bai'



### Assignment 3

Open IGV and select reference hg19. Now load the .bam files of sample_A.sorted, sample_B and sample_C into IGV. You need to select both the .bam files and the .bai files. You can either use IGV on the [cluster](https://www.uppmax.uu.se/support/user-guides/integrative-genomics-viewer--igv--guide/), or you can download the files and start IGV locally on yout computer. 

!!! question

	??? info
		You can click on a read and get more information about the read. If you right click on a read, you have the option of visualizing different features such as mate pairs, or mapping quality.  
		Click on the coverage plot part of a base to get more information on read depth of the base and the allele frequency of the variants found in this position. You can also see which strand they were found on. 


	:question:
	Navigate to the gene ABL1. Look at the reference at the bottom to find where the exons are and where targeted regions might be sequenced. Zoom in closer and navigate to exonic sites. What is the difference between the different .bam files? See how the reads are mapped and also look at the coverage plots at the top of each .bam file. Which sample do you think is a sequenced with a Whole Genome, amplicon and capture strategy?  
         

	:question: 
	Navigate to position 133748283 on this chromosome. There is a variant here in the sample sequenced with the  capture method. You can get a read count and allele frequency  when clicking at the position in the coverage plot.  
        What is the allele frequency and read depth at this position for the capture sequence data? What about the amplicon sequence data? And what about the WGS sample? 


	!!! note 
		Notice how the WGS sample looks very messy, as if most bases carry a variant? Do you have an idea why? Remove the sample from IGV, we will look at it later. 


	:question:
	Navigate to chr9:5070021.  Which gene are we looking at? What variant do you observe at this position in the capture and amplicon sample. How long is the variant? What is the affected sequence? 
     

	Navigate to chr9:5073770. What type of variant do you find here? What is the read depth, and allele frequencies in the different .bam files? 


	:question:
	Navigate to chr5:170837543. Which gene are we looking at? What variant do you observe at this position in the capture and amplicon sample. How long is the variant? What is the affected sequence?

 
	:question:
	Navigate to chr5:170846329-170847798. Look at the sample sequence with a capture method. Make sure to scroll all the way up so you can see the coverage plot. What is the read depth here? How does this compare to the read depth in other areas we looked at. Check the reference, are you looking at a gene? Intronic or extronic region? What do you think is happening here?   


	:question:
	The WGS sample was mapped to hg38. Lets switch the reference accordingly and open the sample again. Do you notice a difference to how it looked like before? Try to find some SNVs, what are their allele frequencies? Mark down a few that you can find along with how many reads support them and how many strands are forward and reverse. Which do you think are real variants and which do you think are artefacts?


	:question: 
	Switch the reference genome again and look at sample_C. Look again at the variant in detail chr9:133748283. Is this an artefact? Why - why not? 


	:question:
	Which sample do you think is somatic, which is germline? 


## VCF

VCF stands for **v**ariant **c**all **f**ormat. A vcf file is used to store variations of the genome encountered in a sample or data set. Each row in a vcf file describes a variant of a specific genomic position in your sample(s). You can either save all sites of the genome in a vcf-file, referred to as an all-sites-vcf or just sites that differ from  the reference.

As bam files, vcf files also contain a header that describes the information found in the file. It also shows which vcf version is used to encode information in this file. You can always check the specific documentation for the vcf version if you do not know how to interpret the data. PDFs of the documentation of the most recent versions of vcf can be found [here](https://github.com/samtools/hts-specs). 

You can filter and edit vcf files using [bcftools](https://samtools.github.io/bcftools/bcftools.html), which functions similarly to samtools. 

### Assignment 4

In this assignment we will look at variants in the vcf file. Some of them we have already looked at in IGV, and you can always compare how the variants look like in the vcf and in the bam file. 

!!! question

	:question:
	Look at the files sample_C.vcf.gz and sample_B.vcf.gz. Are they all-sites-vcf-files? Explain! 


	:question: 
	Search for the variant in chromosome 9 position 133748283 in sample_C.vcf.gz. It's a position in the ABL gene we have looked at before. What is the variant frequency and read depth in the vcf file? How many reads have found either allele? Check the header to understand which tags you are looking for? How do the values compare to the values observed in IGV?


	:question:
	Search for the variant chr9	37025419 in sample_C.vcf.gz. Check the FILTER column and look at the header in sample_C.vcf.gz, what does the flag in the filter field indicate? Look at this position in IGV, how many reads are found on the forward, how many on the reverse strand? How is it for the reference allele? Why do you think this is?    

	:question:
	Using bcftools, figure out how many variants were found on chromosome 9 that passed all filters (FILTER="PASS") in sample_C.vcf.gz. 


If you are done with all the exercises, go check out the rest of the quiz on studium! 

 





