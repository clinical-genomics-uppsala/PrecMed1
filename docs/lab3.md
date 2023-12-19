# Computer lab 3 - Germline Variants
In this computer lab we are going to explore different kinds of germline variants.

## Overview
1. Visualise small and structural germline variants in IGV
2. Find and examine potentially pathogenic germline variants in a number of example cases:
    - Case 1: Pathogenic SNV in TP53 gene
    - Case 2: Patient with hemochromotosis
    - Case 3: Analysis of a Trio using variants and HPO terms
    - Case 4: Structural variant underlying a heritable cancer syndrome
    - Case 5: Repeat Expansion

## Prerequisites
* [IGV](https://igv.org/) - Desktop application or web app
* Shell / terminal / putty
* rsync
* singularity
* bcftools

## Links to resources
* [Genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/)
* [Viewing variants in IGV]()
* [OMIM](https://omim.org/)
* [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/)
* [STRipy STR database](https://stripy.org/database)


### IGV
Open IGV and then open the `NA12878_examples.bam` file. 

### 1. Visulising Germline Variants in IGV
In the regions below there are different type of variants or mutations. For additional information regarding genetic variant types see [genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/).

**Regions**
* chr4:115007570-115010724
* chr6:3578674-3579881
* chr5:148173476-148175052

!!! question
    :question: 
    Determine the type of variants displayed above.
        * Inversion
        * Duplication
        * Deletion

!!! TIP
    * Copy the regions into IGV
    * Zoom in when needed
    * View as pairs
    * Color alignments by -> insert size and pair orientation
    * Sort alignments by -> insert size

### 2. Cases

#### Case1

TP53 mutation

#### Case 2

##### Clinical and Family History

A 55 year old male patient has been experiencing symptoms of lethargy and aches in his joints. Blood tests peformed displayed elevated levels of ferritin. The patient does not have a family history of confirmed hemochromotosis type 1. 

There are a number of ways to obtain a list of gene disease associations. In this case we have a good idea what the disease is and so we will search the Online Mendelian Inheritance in Man (OMIM) catalog for hemochromotosis type 1 and navigate to the molecular genetics section.

!!! questions
    :question: 
    In which gene should we be searching for variants?

Using bcftools or grep extract the variants for the relevant gene and identify any pathogenic mutations. 

    !!! questions
    :question: 
    What is the consequence of the variant?
    What is the impact of the variant?
    What genotype does the sample have and what is the inheritance mode of hemochromotosis type 1?

The CLINSIG field of the VEP annotation can contain multiple entries. Use the variant rs-id to search in [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/) and find more information on the molecular consequences of this variant.

    !!! questions
    :question: 
    What molecular consequences are listed for this variant in the VCF? Is this different from those reported by VEP? Which transcript is reported in VEP and why?

#### Case3

Analysis or rare genetic diseases if often carries out on a trio (proband, mother and father). Here we will analyse a trio VCF of SNPs and INDELs to try and find the causal variant for the following case:

##### Clinical and Family History
A two year-old female presented for a well child check. Her early pediatric milestones were reassuring, she walked shortly after her first birthday and began talking around the same time. However, at the appointment, her mother notes that she is no longer using words that she used to and that she no longer feeds herself and seems less engaged. On examination, her growth parameters are in the normal range, but are notable for minimal growth in head circumference over the last 9 months. She verbalizes but does not make intelligible words and does not reciprocate in verbal exchange. She is an only child and the family history is negative with respect to significant medical problems.

###### HPO Terms
Seizure()
Spasticity()
Postnatal microcephaly()
Intellectual disability profound()

##### Trio Confirmation
When analysing a trio an important quality check is to confirm that the samples fit the expected relatedness. The relatedness between samples are specified in a [PED](https://www.cog-genomics.org/plink2/formats#fam) file called case3.ped. Use the program peddy to perform a quality control on the trio VCF. The peddy results are written in a html report file called **peddy_results.html**. More information on the peddy output can be found in the peddy [documentation](https://peddy.readthedocs.io/en/latest/).


    !!! questions
    :question: 
        1. Do the samples show the expected relatedness and correct sex?
        3. Peddy's sex check uses genotype calls on ChrX to determine sex, can you suggest an alternative method using a BAM file with mapped reads?

##### Using HPO terms  
We are also provided with HPO terms in the file case3_hpo.txt. We will use these HPO terms to try and identify some candidate genes. To do this we will use a tool that takes HPO terms and returns a prioritized list of genes associated with the phenotypes described by the HPO terms. Go to [https://phen2gene.wglab.org/](https://phen2gene.wglab.org/), paste in the hpo terms listed in the **hpo_terms.txt** in the box for HPO IDs and click submit.

Take the Gene that receives the highest ranking in the list and extract any variants in the VCF file from the gene. The trio VCF file has been annotated with gene information and population allele frequency data using VEP, so we can use bcftools or grep to extract the variants from the gene of interest.

    !!! questions
    :question: 
        1. What kind of pathogenic variant is present in this gene and what is its consequence? 
        2. What is the inheritance mode of the variant in this case?
        3. What phenotype on OMIM might fit with this inheritance mode and clincical history?

#### Case4

Here we will examine a VCF with structural variants from a patient 

##### Clinical and Family History
A patient diagnosed with colon cancer and with a family history of colon cancer. Genetic testing to determine if the patient has Lynch Syndrome (hereditary nonpolyposis colorectal cancer).

Here we will use ClinVar's  to obtain a list of genes associated with Lynch Syndrome. We will then use the gene list to narrow down the list of variants in our VCF file. 

```bash

wget -qO - ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/gene_condition_source_id     | cut -f 2,5 | grep "Lynch syndrome" | cut -f1 > lynch_syndrome_genes.txt

```

Use grep to extract variants that overlap the genes listed above.

!!! questions
    :question: 
    1. Which genes are impacted by structural variants?
    2. 


#### Case5

This male patient aged 40 has symptoms that include mild muscule weakness, myotonia, and cataracts. This patient is suspected to have Myotonic dystrophy type 1 (DM1). 

##### Repeat expansion detection

Myotonic dystrophy type 1 is a progressive neuromuscular disease that occurs due to a CTG expansion in the 3' UTR of the DMPK gene. To detect a repeat expansion in DMPK we will use a program called ExpansionHunter. A description and explanation of the VCF produced by Expansion Hunter is available [here](
https://github.com/Illumina/ExpansionHunter/blob/master/docs/06_OutputVcfFiles.md)

All files needed are in the Case5 folder

```bash
    singularity  exec --no-home -B /beegfs-storage/,$PWD:$PWD \
    docker://hydragenetics/expansionhunter:5.0.0 \
    ExpansionHunter --reads case5.bam \
        --reference ${ref_fasta} \
        --threads $(nproc) \
        --variant-catalog ${catalog} \
        --output-prefix case5.expansion_hunter
```

We will then use the tool stranger to annotate the short tandem repeats in the ExpansionHunter VCF. This program adds additional information whether the length of the alleles estimated are pathological. See [here](https://github.com/Clinical-Genomics/stranger?tab=readme-ov-file#output) for further information on the stranger annotated vcf.

```bash
singularity  exec --no-home -B /beegfs-storage/,$PWD:$PWD \
    docker://hydragenetics/stranger:0.8.1 \
    stranger -f ${catalog} case5.expansion_hunter.vcf.gz > case5.stranger_annotated.vcf
```


!!! questions
    :question: 
    1. How many repeats does the patient have in the longer allele and is it pathogenic?
    2. The tri-nucleotide repeat motif for DMPK is CTG. Why does Expansion Hunter report a repeat motif of CAG?
    3. Expansion hunter divided supporting reads into spanning, flanking and in-repeat
    reads. In this case which tyoe of read offers the most support to the expanded allele? 

