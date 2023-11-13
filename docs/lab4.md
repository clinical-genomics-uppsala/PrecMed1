# Computer lab 4 - Somatic variants
In this computer lab we are going to explore different kinds of somatic variants.

## Overview
1. Explore different somatic variants in IGV
2. Look at a gene fusion
3. Run a small somatic pipeline
4. Variant annotations in a vcf file
5. Somatic copy number variations

## Prerequisites
* IGV
* Shell / terminal / putty
* rsync

## Files
Download the lab 4 files to your local computer. Exchange {user} with your UPPMAX username.

```bash
rsync -ah {user}@rackham.uppmax.uu.se:/proj/snic2017-1-1/lab4_files ~/
```

## Somatic variant types
Lets first explore some different types of variant that can occur in a somatic sample. 

### IGV
Open IGV and then open the `RNA-SeraSeq_R.FGFR3.TACC3.bam` file. 

### Variants
In the regions below there are different type of variants or mutations.

**Regions:**

* chr7:140,447,299-140,450,937
* chr2:29,416,286-29,416,459
* chr7:140,432,776-140,436,414
* chr3:41,265,566-41,266,697
* chr2:29,416,005-29,416,044

!!! question
    :question: 
    In the regions below there are variants, determine which variants is an

    * Somatic SNV
    * Germline SNV
    * SNV artifact
    * Insertion
    * Deletion


!!! question
    :question: 
    Determine the exact start position of the variants for all non-artifact variants

!!! question
    :question: 
    Determine the number of supporting reads for all non-artifact variants

!!! question
    :question: 
    Determine the allele frequency for all non-artifact variants

!!! TIP
    * Copy the regions into IGV
    * Zoom in when needed
    * Left click on positions in the coverage track to see support for each allele
    * Right click in the read track and choose Collapsed or Squished to see more of the data

---

### Fusions
Now, lets look at a fusion event between the genes FGFR3 and TACC3.

1. Make a new session in IGV or open a new window.
2. Open the file `RNA-SeraSeq_R.FGFR3.TACC3.bam` in IGV. 
3. Copy and paste both of the following regions into IGV at the same time:
> chr4:1807661-1809661 chr4:1738429-1744429

<br/>
!!! question
    :question:
    Note that this is RNA sequency data from a capture design where only the FGFR3 gene was captured and not TACC3. How come we can see reads in the TACC3 gene anyway given that there is a gene fusion between these two genes?

### Fusion break point
Using the current visualization settings it is hard to see the fusion break point. We should therefore change the settings in IGV so that soft clipped bases are shown. These are bases that does not match to the reference genome and as each mismatch has its own color the reads will look a bit like a rainbow.

1. Open the View menu
2. Choose Preferences
3. Choose the Alignment tab
4. Scroll down and check the box named "Show soft-clipped bases"
5. Click the Save button

<br/>
!!! question
    :question:
    Using the gene track and the first transcript shown for each gene (NM_022965.4 and NM_006342.2) determine between which exons the fusion has occurred.

### Fusion visualization using the Arriba fusion caller
Somatic fusion calling can be performed by a number of different callers. One such caller is caller [Arriba](https://github.com/suhrig/arriba). The nice thing with this caller is that the fusions that they call are visualized in a very nice way.

Open the pdf file `RNA-SeraSeq_R.arriba.fusions.pdf` produced by Arriba. The first fusion in this document is the same one that we previously studied in IGV, FGFR3::TACC3. Here we can see the fusion breakpoint as well as the direction and if it is in-frame. This together with the included protein domains in the fusion product is important to determine if the fusion is clinically relevant.

!!! question
    :question:
    Which of the two proteins are mainly involved in cancer?

!!! question
    :question:
    Why is it clinically relevant to find FGFR3::TACC3 fusions.

<br/>
Scroll further down in the document to find further fusion that Arriba has found. Usually only one true fusion is found per sample but in this case it is a synthetically created sample with many clinically relevant fusions. 

---

## Run somatic pipeline

---

## Variant annotation in vcf
Use grep to investigate the variants found earlier.

---

## Copy number variation (CNV)
In cancer cells there is quite common with large and many copy number alterations, especially in certain solid tumors like lung cancers. Here, we are going to look at how it can look in a few solid cancer samples. 

### Normal sample
However, lets start with how it looks in a normal sample without alterations. Open `CNV/normal_sample.cnv.html` in a browser. The first plot show the zoomed-in view of the data and the second plot and overview of the entire genome. There is also a table to the right with potentially clinically relevant CNVs. Consider the tips below and play around with the data in the CNV html report.

!!! TIP
    * Click on a chromosome in the overview to show it in the top plot.
    * In the top plot, left-click and drag to zoom in.
    * In the top plot, left-click and to zoom out to chromosome view.

!!! NOTE
    Log2ratio is the copy number in the sample and is calculated as a ratio to the expected number of copies (2) and then log2 is applied. A normal copy number of 2 is therefore log2(2/2)=log2(1)=0. One extra copy is log2(3/2)=log(1,5)=0.6 and one lost copy is log2(1/2)=log(0,5)=-1. However, this only applies if the tumor content is 100%. Otherwise, the effect of a somatic CNV will be smaller as only a subset of the cells are affected.

!!! NOTE
    The VAF-plots show the allele frequencies of heterozygous germline SNPs in the sample. When there is a deletion or duplication the allele frequencies will increase or decrease depending on the allele. This is used as additional evidence of amplifications and deletions.

### Sample with CNVs
Now, lets consider a sample with chromosomal alterations: `CNV/tumor.cnv.html`. 

!!! question
    :question:
    How many chromosomes have deletions?

!!! question
    :question:
    How many chromosomes have duplications?

!!! question
    :question:
    Look at VAF-figure in chromosome 3 as an example. Why is the signal (the size of the separation) stronger for a deletion than an duplication?

### Sample with clinically relevant amplification
In this sample (`CNV/EGFR_amp.cnv.html`) there is an amplification of EGFR. Cancers with this amplification are often resistant to tyrosine kinase inhibitors and can therefore affect the treatment of the patient. 

Find the EGFR amplification and zoom in on the gene.

!!! TIP
    * When there are really high amplifications the data is sometimes of the scale. Click the box "Zoom to data extent" to see all the data.

!!! question
    :question:
    What is the approximate log2 ratio of the EGFR mutation? 

!!! question
    :question:
    How many copies does this log2 ratio represent?

!!! question
    :question:
    In the table, what is the actual copy number reported? (The table reports the copy number after taking tumor content into account.)


### Sample with clinically relevant deletion
In the sample `CNV/CDKNA_B_del.cnv.html` there is a homozygous deletion of the gene CDKN2A and CDKN2B. This is a diagnostic marker is some cancers associated with poor prognosis. 

Find the CDKN2A and CDKN2B deletion. 

!!! question
    :question:
    Based on the copy number plot and the VAF-plot it seems that most of chromosome 10 has only one copy in the tumor (hemizygous deletion). Motivate, based on both the log2 ratio and the VAF plot why the region overlapping CDKN2A and CDKN2B are a homozygous deletion in the tumor (both alleles deleted).

### Final / extra CNV question
!!! question
    :question:
    Consider a hypothetical case were one chromosome has two copies according to the log2 ratio plot at the same time as there is a clear separation in the VAF-signal plot (all SNPs far away from 50% in allele frequency). What has happened to the chromosome in this tumor to explain these data?

---