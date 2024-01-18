# Computer lab 3 - Germline Variants
In this computer lab we are going to explore different kinds of germline variants.

## Overview
1. Visualise structural germline variants in IGV
2. Find and examine potentially pathogenic germline variants in a number of example cases:
    - Case 1: Pathogenic SNV in TP53 gene
    - Case 2: Patient with hemochromotosis
    - Case 3: Analysis of a Trio using variants and HPO terms
    - Case 4: Structural variant underlying a heritable cancer syndrome
    - Case 5: Repeat Expansion detection

## Prerequisites 
All of the below are installed or available through the module system on rackham/snowy.

* [IGV](https://igv.org/) - Desktop application or web app  
* Shell / terminal / putty  
* rsync  
* singularity  
* bcftools  


## Files
Login to [rackham](https://www.uppmax.uu.se/support/user-guides/rackham-user-guide/) on UPPMAX.
Download the lab 3 files to your home directory on rackham (UPPMAX).

```bash
cp -r /proj/uppmax2024-2-1/nobackup/lab_files/lab3_germline/ ~/
cd lab3_germline
```

## Interactive jobs on Uppmax

In this lab all analysis will be performed in an interactive job on the snowy cluster. The following command can be used to launch an interactive session for 4 hours with 1 core allocated.

```bash
interactive -t 04:00:00 -n 1 -p core -M snowy -A uppmax2024-2-1
```

## Links to resources
* [Genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/)
* [IGV desktop Documentation](https://igv.org/doc/desktop/)
* [VEP glossary](https://www.ensembl.org/info/website/glossary.html)
* [OMIM](https://omim.org/)
* [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/)
* [STRipy STR database](https://stripy.org/database)

## 1. Visualising Germline Variants in IGV
The visual inspection of the sequencing data supporting a germline variant is an important part of a clinical bioinformaticians work. Here we will use IGV to visualise some structural variants.

The reads in the BAM file we will used have been mapped to the **hg38** reference genome, so make sure to switch to this genome version in IGV.

Open IGV and then open the `NA12878_examples.bam` file in the `sv_examples` folder. 

In the regions below there are different types of structural variants.

- For additional information regarding genetic variant types see [genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/).
- For additional information on interpreting read mapping in IGV see the [IGV documentation](https://igv.org/doc/desktop/#UserGuide/tracks/alignments/paired_end_alignments/#insert-size) on paired end alignments.

### Regions
* chr4:115005320-115012973
* chr20:43695763-43697956
* chr1:197786656-197790287

!!! question "Question 1"
    :question: 
    Determine the type of variants displayed above

    - Inversion
    - Tandem Duplication
    - Deletion

!!! TIP
    * Copy the regions into IGV
    * Zoom in or out as needed 
    * View as pairs
    * Color alignments by -> insert size and pair orientation
    * Sort alignments by -> insert size

---
## 2. Cases

### Case1 - Pathogenic SNV in TP53 gene

#### VCF with VEP annotations
Here we will take an example VCF file with a pathogenic variant in the TP53 gene. This case will serve as an example on how to filter variants in a vcf file and how to extract information about the variants. **There are no questions for this case**. 

Here you are given a VCF file called `case1.vep_annotated.vcf.gz` containing SNVs and INDELs. The VCF has been annotated with variant effect information using the [Variant Effect Predictor](https://www.ensembl.org/info/docs/tools/vep/index.html) tool to calculate the [variant consequences](https://www.ensembl.org/info/genome/variation/prediction/predicted_data.html) of each variant in the VCF. The variant information added by VEP is located in the INFO field of starting with "CSQ=".

We can extract information about what the various fields in the VEP annotations are using a grep search of the VCF header:

```bash

zgrep  ^# case1.vep_annotated.vcf.gz | grep CSQ

```

Given the information above a simple grep search for TP53 and 'pathogenic' will help us narrow down the search to two variants.

```bash

zgrep -w TP53 case1.vep_annotated.vcf.gz | grep pathogenic

```

Examine the output above and see if you can identify the information on the consequence and impact of each variant.

#### Using bcftools

Using grep is a simple and quick way to extract information from a VCF. However, there are many more specialist tools one can use with VCF files. Here we will use the popular bcftools program which has many options available for filtering vcf files and has a plugin for handling the VEP formatted information. 

The bcftools program is available on uppmax through the module system and can be loaded as follows:

```bash
module load bioinfo-tools bcftools/1.19
```

One loaded we can list all the available VEP subfields from the CSQ INFO field

```bash
bcftools +split-vep case1.vep_annotated.vcf.gz -l 
```

Here we will take a number of steps to filter the VCF file and we will make use of pipes to perform each step sequentially. Here we visualise the VCF output of each step with `less -S`. Just type `q` to exit `less` in the terminal.

1. Exclude all records in the VCF that do not have 'PASS' in the FILTER column
    ```bash 
    bcftools view -f PASS case1.vep_annotated.vcf.gz | less -S
    ```
2. Exclude common variants, as we are only interested in the rare variants. Here we will use the [MAX_AF subfield](https://www.ensembl.org/info/docs/tools/vep/script/vep_options.html#opt_max_af) and a [filtering expression](https://samtools.github.io/bcftools/bcftools.html#expressions) from bcftools.
    ```bash 
    bcftools view -f PASS case1.vep_annotated.vcf.gz |  bcftools +split-vep -c MAX_AF:Float -e "MAX_AF>0.1" | less -S
    ```
3. Keep only variants in the TP53 gene.  
    ```bash 
    bcftools view -f PASS case1.vep_annotated.vcf.gz |  bcftools +split-vep -c MAX_AF:Float,SYMBOL -e "MAX_AF>0.1 | SYMBOL!='TP53'" | less -S
    ```

Using bcftools also allows the output  to be formatted in a more human readable way where a subset of columns that are of most interest can be extracted (e.g., Consequence and CLIN_SIG can be extracted to look for evidence of pathogenicity) :
```bash 
bcftools view -f PASS case1.vep_annotated.vcf.gz |  bcftools +split-vep -c MAX_AF:Float,SYMBOL -e "MAX_AF>0.1 | SYMBOL!='TP53'" -f "%CHROM %POS %SYMBOL %Consequence %IMPACT %Protein_position %Amino_acids %CLIN_SIG [%GT ]\n"
```


### Case 2 - Patient with hemochromotosis

#### Clinical and Family History

A 55 year old male patient has been experiencing symptoms of lethargy and aches in his joints. Blood tests performed displayed elevated levels of ferritin. The patient does not have a family history of confirmed hemochromotosis type 1 which is a disease characterised by an accumulation of too much iron in the body.

#### Searching OMIM
There are a number of ways to obtain a list of gene disease associations. In this case we have a good idea what the disease is and so we will search the Online Mendelian Inheritance in Man ([OMIM](https://omim.org/)) catalog for hemochromotosis type 1 and navigate to the molecular genetics section.

!!! question "Question 2"
    :question: 
    In which gene should we be searching for variants?

#### Extracting variants
Using `bcftools`, or `grep`, extract the variants from the gene identified above from the  `case2.vep_annotated.vcf.gz` file case 2 folder. You can filter using the bcftool commands from steps 1 and steps3 from case1 above.

!!! question "Question 3"
    :question:
    Do you see any variants that have evidence of pathogenicity? What is the consequence and the impact of the variant that has evidence?
!!! question "Question 4"
    :question:
    What is the the MAX_AF for the variant?
!!! question "Question 5"
    :question:
    What genotype does the sample have and what is the [mode of inheritance](https://www.ncbi.nlm.nih.gov/books/NBK115561/) of hemochromotosis type 1?  
---
### Case3 - Analysis of a Trio using variants and HPO terms

Analysis of rare genetic diseases is often carried out on a trio (proband, mother and father). Here we will analyse a trio VCF of SNVs and INDELs to try and find the causal variant for the following case:

#### Clinical and Family History
A two year-old female presented for a welfare check. Her early pediatric milestones were reassuring, she walked shortly after her first birthday and began talking around the same time. However, at the appointment, her mother notes that she is no longer using words that she used to and that she no longer feeds herself and seems less engaged. On examination, her growth parameters are in the normal range, but are notable for minimal growth in head circumference over the last 9 months. She verbalizes but does not make intelligible words and does not reciprocate in verbal exchange. She is an only child and the family history is negative with respect to significant medical problems.

##### HPO Terms
* [Seizure](https://hpo.jax.org/app/browse/term/HP:0001250)
* [Spasticity](https://hpo.jax.org/app/browse/term/HP:0001257)
* [Postnatal microcephaly](https://hpo.jax.org/app/browse/term/HP:0005484)
* [Intellectual disability](https://hpo.jax.org/app/browse/term/HP:0002187)

#### Trio Confirmation
When analysing a trio an important quality check is to confirm that the samples fit the expected relatedness. The relatedness between samples are specified in a [PED](https://www.cog-genomics.org/plink2/formats#fam) file called `case3.ped`. Use the program peddy to perform a quality control on the trio VCF. The peddy results are written in a html report file called **`peddy_results.html`**. More information on the peddy output can be found in the peddy [documentation](https://peddy.readthedocs.io/en/latest/).

All files needed are in the Case3 folder and the commands below assume that you are in the case3 folder.

```bash
    singularity  exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/nobackup/singularity_images_lab3/peddy_0.4.8.sif \
    python -m peddy --sites hg38 \
    --prefix peddy_results \
    case3.vep_annotated.vcf.gz case3.ped

```

!!! question "Question 6"
    :question: 
    View the peddy_results.html file by copying it to your laptop and viewing it in your web browser. Do the samples show the expected relatedness and correct sex?
        
!!! question "Question 7"
    :question: 
    How does peddy check the sex of a sample?
    ??? tip "Hint"
        Consult the [peddy documentation](https://peddy.readthedocs.io/en/latest/#sex-check)

#### Using HPO terms to prioritise genes
We are also provided with HPO terms in the file `case3_hpo.txt`. We will use these HPO terms to try and identify some candidate genes. To do this we will use a tool that takes HPO terms and returns a prioritized list of genes associated with the phenotypes described by the HPO terms. Go to [https://phen2gene.wglab.org/](https://phen2gene.wglab.org/), paste in the hpo terms listed in the **`hpo_terms.txt`** in the box for HPO IDs and click submit.

Take the Gene that receives the highest ranking in the list and extract any variants in the VCF file from the gene. Filtering this VCF file with  bcftools will take ~1 minute before any output is written.

!!! question "Question 8"
    :question: 
    What kind of pathogenic variant is present in this gene and what is its consequence? 

!!! question "Question 9"
    :question:   
    What is the inheritance mode of the variant in this case?


---
### Case4 - Structural variant underlying a heritable cancer syndrome

Here we will examine a VCF with structural variants from a patient 

#### Clinical and Family History
A patient diagnosed with colon cancer and with a family history of colon cancer. Genetic testing is performed to determine if the patient has Lynch Syndrome (hereditary nonpolyposis colorectal cancer). The analysis of SNVs and INDEL variants did not reveal any likely pathogenic variants. Therefore structural variant calling was performed using the structural variant (SV) caller [Manta](https://github.com/Illumina/manta) which combines paired and split-read evidence to call SVs.

#### Variant filtration
Take a look inside the file called `case4_manta.vep_annotated.vcf.gz` using `less -S`. You can see that this file is annotated with VEP and has information from ClinVar and Genome Aggregation Database (gnomAD) SV v4.0. We will perform three filtration steps to narrow down the list of SVs:

1. Retain only records with a PASS in the filter field.  
2. Look for variants that overlap with a set of candidate genes in the ```lynch_syndrome_genes.bed```. 
3. Exclude variants with an allele frequency in Gnomad SV 4.0 > 0.1 (called gnomad_AF in the VCF VEP annotation field).  

!!! note
    For step 2 you can use the ```-R lynch_syndrome_genes.bed``` with `bcftools view` command to extract variants that overlap with the genes listed in the bed file.


The genes associated with a disease can be obtained from a number of sources. Here the genes listed for step 2 above were obtained from ClinVar's list of genes associated with Lynch Syndrome. 

```bash

wget -qO - ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/gene_condition_source_id   | cut -f 2,5 | grep "Lynch syndrome" | cut -f1 | sort | uniq 

```

Use the filtered vcf generated from Steps 1-3 above and the `case4_sv.bam` file to answer the following questions:


!!! question "Question 10"
    :question: 
    Which of the Lynch Syndrome genes are affected by a structural variant?

!!! question "Question 11"
    :question:
    What type of structural variant affects this gene and how large is it?

!!! question "Question 12"
    :question: 
    Using the `case4_sv.bam` file, view the Structural variant in IGV. Do the patterns of insert size and read coverage support for the structural variant indicate that the call made by Manta is correct?
    ??? tip "Hint"
        View as pairs option in IGV can help visualise the effect of the SV on insert size

---
### Case5 - Repeat expansion detection

Next we will take a look at the difficult problem of calling expansions of short tandem repeats (STRs) with whole genome Illumina paired-end data.

#### Clinical and Family History
This male patient, age 40, has symptoms that include mild muscle weakness, myotonia, and cataracts. This patient is suspected to have Myotonic dystrophy type 1 (DM1). Myotonic dystrophy type 1 is a progressive neuromuscular disease that occurs due to a CTG expansion in the 3' UTR of the DMPK gene. 

#### Repeat expansion genotyping
To detect a repeat expansion in DMPK we will use a program called ExpansionHunter. A description and explanation of the VCF produced by ExpansionHunter is available [here](
https://github.com/Illumina/ExpansionHunter/blob/master/docs/06_OutputVcfFiles.md). The `case5.bam` file contains reads mapped to a region of chr19 that contains the DMPK gene.

All files needed are in the Case5 folder and the commands below assume that you are in the case5 folder.

```bash
singularity exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/nobackup/singularity_images_lab3/expansionhunter_5.0.0.sif \
    ExpansionHunter --reads case5.bam \
    --reference homo_sapiens.fasta  \
    --variant-catalog variant_catalog_hg38.json  \
    --output-prefix case5.expansion_hunter
```


Before we proceed further we need to sort and index the small bam file produced by ExpansionHunter using samtools available through the module system on Uppmax.

```
module load bioinfo-tools
module load samtools

samtools sort -O BAM  case5.expansion_hunter_realigned.bam -o case5.expansion_hunter_realigned.sorted.bam
samtools index case5.expansion_hunter_realigned.sorted.bam

```

To visualise the read support for the ExpansionHunter results we will use a program called REViewer which works with the small BAM file produced by expansionHunter. More information on REViewer is available in this [blog post](https://www.illumina.com/science/genomics-research/articles/reviewer-alignments-short-reads-long-repeat.html) from illumina and in the [REViewer publication](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-022-01085-z).
REViewer requires the sorted `case5.expansion_hunter_realigned.sorted.bam`, the `case5.expansion_hunter.vcf` output by ExpansionHunter and the variant catalog file to produce a plot for the DMPK locus. 

```bash
singularity exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/nobackup/singularity_images_lab3/reviewer_0.2.7.sif \
    REViewer --reads case5.expansion_hunter_realigned.sorted.bam \
    --vcf case5.expansion_hunter.vcf \
    --reference homo_sapiens.fasta \
    --catalog  variant_catalog_hg38.json \
    --locus DMPK \
    --output-prefix case5.reviewer

```

The image output by REViewer is in SVG format. This is best viewed in a web browser such as firefox or Google Chrome. You can download 
the file called `case5.reviewer.DMPK.svg` and open it with your web browser. Examine the reviewer plot to understand the division of the supporting reads into the spanning, flanking and in-repeat categories, and to assess the coverage for each allele.

We will now use the tool [Stranger](https://github.com/Clinical-Genomics/stranger) to annotate the short tandem repeats in the ExpansionHunter VCF. This program adds additional information whether the length of the alleles estimated by ExpansionHunter are pathological. See [here](https://github.com/Clinical-Genomics/stranger?tab=readme-ov-file#output) for further information on the stranger annotated vcf.

```bash
singularity exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/nobackup/singularity_images_lab3/stranger_0.8.1.sif \
    stranger -f variant_catalog_hg38.json case5.expansion_hunter.vcf > case5.stranger_annotated.vcf

```

Use the `case5.stranger_annotated.vcf` file to answer the following questions:

!!! question  "Question 13"
    :question: 
    How many repeats does the patient have in each allele?

!!! question  "Question 14"
    :question:  Are the both DMPK alleles above the pathogenic threshold?

!!! question  "Question 15"
    :question:
    Expansion hunter divides supporting reads into spanning, flanking and in-repeat
    reads. How many spanning, flanking and in-repeat reads support the longer expanded allele? 


 A good resource for staying updated on pathogenic expansion repeats is the [STRipy STR database](https://stripy.org/database). Go to the DMPK entry to see information on this repeat expansion and the frequency of different repeat lengths in gnomAD.   

!!! question  "Question 16"
    :question: The tri-nucleotide repeat motif for DMPK in the literature is given as CTG. Why does Expansion Hunter report a repeat motif of CAG?  

---
## References 

Case 3 comes from the case 5 patient scenario on the [genebreaker website](http://genebreaker.cmmt.ubc.ca/patient_scenarios). 