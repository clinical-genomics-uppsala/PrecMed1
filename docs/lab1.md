# Sequencing data formats and visualization 
In this computer lab we will talk about and have a look at the most common file formats used for NGS sequencing data. 

## FASTQ
The FASTQ file format is a progression of the FASTA file format, developed for NGS data.  

fastq files often contain multiple reads that are described in four rows:
 
1. sequence identifier, row starts with @ 

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


### Assignment 1 
!!! question 
	In `PATH` you will find an example of a fastq file generated with a NovaSeq sequencing machine: ``FILENAME.fq`
    

	:question:
	Have a look at the quality scores, what do you think is happening here? 
     

	:question:
	What Phred score value do the symbols you see correspond to? Here is a link to an [ASCII table](https://en.wikipedia.org/wiki/ASCII#Printable_characters). We are counting from 33 in the decimal system. 

## BAM
Output from mapping is stored in .sam/.bam/.cram files.
These files are tab-delimited and container a header, where header rows start with an @, and the alignment data, with one row per aligned sequence. 


### Assignment 2 

In your terminal look and compare the bam files in the directory. You can use samtools to explore bam files. Have a look at the [documentation](https://www.htslib.org/doc/samtools-view.html). We first want to explore the bam header, so find the command flag to include or show the header in your output. These files are big so make sure to pipe into `less`. 

!!! question
	

	:question:
	Notice that there are tags in the header. You can check what these mean in the [samtools manual](https://samtools.github.io/hts-specs/SAMv1.pdf) In the file HD892_df_C which programs have been used to create the bam file? 


### Assignment 3

Open IGV and select reference hg19. Now load the files HD829_df_A, HD829_df_C and in IGV. You need to select both the .bam files and the .bai files.
!!! question


	:question:
	Navigate to the gene ABL1. Look at the reference at the bottom to find where the exons are and where targeted regions might be sequenced. Zoom in closer and navigate to exonic sites. What is the difference between the different .bam files? See how the reads are mapped and also look at the coverage plots at the top of each .bam file. Which sample do you think is a sequenced with a Whole Genome, amplicon and capture strategy?  


	:question: 
	Navigate to position 133748283. We know that there is a variant here in the sample sequenced with an amplicon or capture method. You can get a read count and allele frequency  when clicking at the position in the coverage plot.  What is the allele frequency and read depth at this position for the capture sequence data? What is about the amplicon sequence data? And what about the WGS sample? 


	:question:
	Navigate to chr9:5070021.  Which gene are we looking at? What variant do you observe at this position in the capture and amplicon sample. How long is the variant? What is the affected sequence? 
     

	Navigate to chr9:5073770. What type of variant do you find here? What is the read depth, and allele frequencies in the different .bam files? 


	:question:
	Navigate to chr5:170837543. Which gene are we looking at? What variant do you observe at this position in the capture and amplicon sample. How long is the variant? What is the affected sequence?

 
	:question:
	Navigate to chr5:170846329-170847798. Look at the sample sequence with a capture method. Make sure to scroll all the way up so you can see the coverage plot. What is the read depth here? 


	??? info
		You can click on a read and get more information about the read. If you right click on a read, you have the option of visualizing different features such as mate pairs, or mapping quality. 

##  VCF 
