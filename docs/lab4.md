# Computer lab 4 - Somatic variants
In this computer lab we are going to explore different kinds of somatic variants.

## Overview
1. Explore different somatic variants in IGV
2. Look at a gene fusion
3. Run a small somatic pipeline
4. Variant annotations in a vcf file
5. Somatic copy number variations

## Prerequisites
* [IGV](https://igv.org/) - Desktop application or web app
* Shell / terminal / putty
* rsync

## Files
Login to [rackham](https://www.uppmax.uu.se/support/user-guides/rackham-user-guide/) on UPPMAX.
Download the lab 4 files to your home directory on rackham (UPPMAX).

```bash
cp -r /proj/uppmax2024-2-1/nobackup/lab_files/lab4_somatic/ ~/
cd lab4_somatic
```

## Links to resources
* [Genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/)
* [The role of fusion genes in cancer](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7931065/)
* [Fusion caller: Arriba](https://arriba.readthedocs.io/en/latest/)
* [CNV caller: CNVkit](https://cnvkit.readthedocs.io/en/stable/pipeline.html)
* [CNVkits explanation of log2 ratio](https://cnvkit.readthedocs.io/en/stable/calling.html)

## Somatic variant types
Lets first explore some different types of variant that can occur in a somatic sample. 

### IGV
Open IGV and then open the `RNA-SeraSeq_R.FGFR3.TACC3.bam` file, make sure that the reference genome is set to *Human hg19*. 

### Variants
In the regions below there are different type of variants or mutations. For additional information regarding genetic variant types see [genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/).

**Regions:**

* chr7:140453022-140453248
* chr2:29416330-29416416
* chr7:140432776-140436414
* chr3:41265991-41266273
* chr2:29416005-29416044

!!! question "Question 1"
    :question: 
    In the regions above there are variants, determine which variant is of which type (there can be more than one)

    * SNV artifact
    * Somatic SNV
    * Germline SNV
    * Deletion
    * Insertion


!!! question "Question 2"
    :question: 
    Determine the exact start position of the variants for all non-artifact variants

!!! question "Question 3"
    :question: 
    Determine the number of supporting reads for all non-artifact variants

!!! question "Question 4"
    :question: 
    Determine the allele frequency for all non-artifact variants
    <!-- Kanske ngn kommentar om IGV AF och varfor den inte alltid ar sa exakt. typ: -->
    ??? note
        IGV uses all reads available in the bamfile, no filtering is applied. Therefor the calculated "raw" allele frequency from IGV does not always correlate with the AF found in vcf-files since callers filters and weights reads before calculating AF.
    

!!! TIP
    * Copy the regions into IGV
    * Zoom in when needed
    * Left click on positions in the coverage track to see support for each allele
    * Right click in the read track and choose Collapsed or Squished to see more of the data

---

### Fusions
Now, lets look at a fusion event between the genes FGFR3 and TACC3. The role of fusions in cancer is summarized in this [scientific paper](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7931065/).

1. Make a new session in IGV or open a new window.
2. Open the file `RNA-SeraSeq_R.FGFR3.TACC3.bam` in IGV. 
3. Copy and paste both of the following regions into IGV at the same time:
> chr4:1807661-1809661 chr4:1738429-1744429

<br/>
!!! question "Question 5"
    :question:
    This is RNA sequency data from a capture design where only the FGFR3 gene was captured and not TACC3. How come we can see reads in the TACC3 gene anyway given that there is a gene fusion between these two genes?

### Fusion break point
Using the current visualization settings it is hard to see the fusion break point. We should therefore change the settings in IGV so that soft clipped bases are shown. These are bases that does not match to the reference genome and as each mismatch has its own color the reads will look a bit like a rainbow.

1. Open the View menu
2. Choose Preferences
3. Choose the Alignment tab
4. Scroll down and check the box named "Show soft-clipped bases"
5. Click the Save button

<br/>
!!! question "Question 6"
    :question:
    Using the gene track and the first transcript shown for each gene (NM_022965.4 and NM_006342.2) determine between which exons the fusion has occurred.

### Fusion visualization using the Arriba fusion caller
Somatic fusion calling can be performed by a number of different callers. One such caller is caller [Arriba](https://arriba.readthedocs.io/en/latest/). The nice thing with this caller is that the fusions that they call are [visualized](https://arriba.readthedocs.io/en/latest/visualization/) in a very clear and informative way. 

Open the pdf file `RNA-SeraSeq_R.arriba.fusions.pdf` produced by Arriba. The first fusion in this document is the same one that we previously studied in IGV, FGFR3::TACC3. Here we can see the fusion breakpoint as well as the direction and if it is in-frame. This together with the included protein domains in the fusion product is important to determine if the fusion is clinically relevant.

!!! question "Question 7"
    :question:
    What role does the ongogenic FGFR3::TACC3 fusions play in cancer?
    
    For example, the function of the fused protein, clinical target and drugs, prevalence, etc.

<br/>
Scroll further down in the document to find remaining fusions that Arriba has found. Usually only one true fusion is found per sample but in this case it is a synthetically created sample with many clinically relevant fusions. 

---

## Run somatic pipeline

### Setup environment
```sh
# Enter pipeline folder
cd ~/lab4_somatic/pipeline

# Setup environment
python3.9 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
```

### Run
```sh
# Make sure you are in the pipleine folder: ~/lab4_somatic/pipeline
# Make sure that environment is active.
# NOTE: not required if you followed the steps in previous section
# and still have that terminal session active
source venv/bin/activate

# load slurm library
module load slurm-drmaa

# Run pipeline
snakemake -s Snakefile --profile profile
```

# The output will end up in folder named analysis
---

## Variant annotation in vcf
Use the `grep` command, in the terminal, on the vcf-file `HD832_T.filtered.vcf` to investigate the variants found earlier. 

Investigate the annotation of the SNV found in the region chr7:140453022-140453248. 

!!! question "Question 8"
    :question:
    What effect does the variant have on the protein sequence?

!!! question "Question 9"
    :question:
    Does it seem to be clinically relevant? Motivate!

---

## Copy number variation (CNV)
In cancer cells there is quite common with large and numerous copy number alterations, especially in certain solid tumors like lung cancers. Here, we are going to look at how it can look in a few solid cancer samples. CNVs are called by cancer specific callers like [CNVkit](https://cnvkit.readthedocs.io/en/stable/pipeline.html).

### Normal sample
However, lets start with how it looks in a normal sample without alterations. Open `CNV/normal_sample.cnv.html` in a browser. The first plot show the zoomed-in view of the data and the second plot an overview of the entire genome. There is also a table to the right with potentially clinically relevant CNVs. Consider the tips below and play around with the data in the CNV html report.

!!! TIP
    * Click on a chromosome in the overview to show it in the top plot.
    * In the top plot, left-click and drag to zoom in.
    * In the top plot, left-click to zoom out to chromosome view.

!!! NOTE
    Log2ratio is the copy number in the sample and is calculated as a ratio to the expected number of copies (diploid genome = 2) and then $ \log_2 $ is applied. A normal copy number of 2 is therefore $ \log_2(\frac {2}{2})=\log_2(1)=0 $. 
    
    If the sample instead have an extra copy then $ \log_2(\frac {3}{2})=\log_2(1,5)=0.6 $ 

    Or one lost copy is $ \log_2(\frac {1}{2})=\log_2(0,5)=-1 $.  

    However, this only applies if the tumor content is 100%. Otherwise, the effect of a somatic CNV will be smaller as only a subset of the cells are affected. See further [CNVkits explanation of log2ratio](https://cnvkit.readthedocs.io/en/stable/calling.html).

!!! NOTE
    The VAF-plots show the allele frequencies of heterozygous germline SNPs in the sample. When there is a deletion or duplication the allele frequencies will increase or decrease depending on the allele. This is used as additional evidence of amplifications and deletions. Sometimes the genome can have two copies of the same allele and the deletion will only 
    show up in the VAF-plot. This is called a [copy number neutral loss of heterozygozity](https://en.wikipedia.org/wiki/Loss_of_heterozygosity).

### Sample with CNVs
Now, lets consider a sample with large chromosomal alterations: `CNV/tumor.cnv.html`. 

!!! question "Question 10"
    :question:
    How many chromosomes have deletions?  (Always take special care when looking at the X chromosome)

!!! question "Question 11"
    :question:
    How many chromosomes have duplications?

!!! question "Question 12"
    :question:
    Look at the VAF-figure in chromosome 3 as an example. Why is the signal (the size of the separation between dots) stronger for a deletion than a duplication?

### Sample with clinically relevant amplification
In this sample (`CNV/EGFR_amp.cnv.html`) there is an amplification of EGFR. Cancers with this amplification are often resistant to tyrosine kinase inhibitors and can therefore affect the treatment of the patient. 

Find the EGFR amplification and zoom in on the gene.

!!! TIP
    When there are really high amplifications the data is sometimes outside the default scale. Click the box "Zoom to data extent" to see all the data.

!!! question "Question 13"
    :question:
    What is the approximate log2 ratio, based on the plot, of the EGFR mutation?

!!! question "Question 14"
    :question:
    How many copies does this log2 ratio represent?

!!! question "Question 15"
    :question:
    The table reports the copy number after taking tumor content into account. What is the actual reported copy number estimated in the tumor?


### Sample with clinically relevant deletion
In the sample `CNV/CDKNA_B_del.cnv.html` there is a homozygous deletion of the gene CDKN2A and CDKN2B. This is a diagnostic marker in some cancer types, associated with poor prognosis. 

Find the CDKN2A and CDKN2B deletion. 

!!! question "Question 16"
    :question:
    Based on the copy number plot and the VAF-plot it seems that most of chromosome 10 has only one copy in the tumor (hemizygous deletion). Motivate, based on both the log2 ratio and the VAF plot why the region overlapping CDKN2A and CDKN2B are a homozygous deletion in the tumor (both alleles deleted).

### Extra CNV questions
!!! question
    :question:
    Consider a hypothetical case were one chromosome has two copies according to the log2 ratio plot at the same time as there is a clear separation in the VAF-signal plot (all SNPs far away from 50% in allele frequency). What has happened to the chromosome in this tumor to explain these data?

!!! question
    :question:
    Consider another hypothetical case were the chromosome with log2 ratio of 0 has a clear separation in the VAF-signal plot while there are chromosomes with both higher and lower log2 ratios that have no VAF-signal. What has happened to the genome in this tumor to explain these data?

