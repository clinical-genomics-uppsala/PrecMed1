# Computer lab 4 - Somatic variants
In this computer lab we are going to explore different kinds of somatic variants.

## Prerequisites
* IGV
* Shell / terminal
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

## Run somatic pipeline

## Variant annotation in vcf
Use grep to investigate the variants found earlier.

---