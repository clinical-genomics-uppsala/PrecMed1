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

In `PATH` you will find an example of a fastq file generated with a NovaSeq sequencing machine: ``FILENAME.fq`

1. Have a look at the quality scores, what do you think is happening here? 

2. What Phred score value do the symbols you see correspond to? Here is a link to an [ASCII table](https://en.wikipedia.org/wiki/ASCII#Printable_characters). We are counting from 33 in the decimal system. 

## BAM
## VCF 
