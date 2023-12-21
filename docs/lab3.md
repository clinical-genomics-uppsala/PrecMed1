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

## Links to resources
* [Genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/)
* [IGV desktop Documentation](https://igv.org/doc/desktop/)
* [VEP glossary](https://www.ensembl.org/info/website/glossary.html)
* [OMIM](https://omim.org/)
* [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/)
* [STRipy STR database](https://stripy.org/database)

### 1. Visualising Germline Variants in IGV
The visual inspection of the sequencing data supporting a germline variant is an important part of a clinical bioinformatics. Here we will use IGV to visualise some structural variants.

Open IGV and then open the `NA12878_examples.bam` file in the sv_examples folder. 

In the regions below there are different types of structural variants. For additional information regarding genetic variant types see [genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/). For additional information on interpreting read mapping in IGV see the [IGV documentation](https://igv.org/doc/desktop/#UserGuide/tracks/alignments/paired_end_alignments/#insert-size) on paired end alignments.

**Regions**
* chr4:115,005,320-115,012,973
* chr20:43,695,763-43,697,956
* chr1:197786656-197790287

!!! question
    :question: 
    Determine the type of variants displayed above.
        * Inversion  
        * Tandem Duplication
        * Deletion  

!!! TIP
    * Copy the regions into IGV  
    * Zoom in or out as needed  
    * View as pairs  
    * Color alignments by -> insert size and  pair   
    * Sort alignments by -> insert size  


### 2. Cases

#### Case1

TODO: add example instructions to show how bcftools or grep can be used to extract VEP info for a  TP53 mutation in a vcf

#### Case 2

##### Clinical and Family History

A 55 year old male patient has been experiencing symptoms of lethargy and aches in his joints. Blood tests performed displayed elevated levels of ferritin. The patient does not have a family history of confirmed hemochromotosis type 1. 

There are a number of ways to obtain a list of gene disease associations. In this case we have a good idea what the disease is and so we will search the Online Mendelian Inheritance in Man (OMIM) catalog for hemochromotosis type 1 and navigate to the molecular genetics section.

    !!! questions
        :question: 
        In which gene should we be searching for variants?

Using bcftools, or grep, extract the variants from the gene identified above in the vcf file inside the case 2 folder. 

    !!! questions
    :question: 
    1. What is the consequence of the variant?
    2. What is the impact of the variant?
    3. What genotype does the sample have and what is the inheritance mode of hemochromotosis type 1?

The CLINSIG field of the VEP annotation can contain multiple entries. Use the variant rs-id to search in [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/) and find more information on the molecular consequences of this variant.

    !!! questions
    :question: 
    What molecular consequences are listed for this variant in ClinVar?  

#### Case3

Analysis of rare genetic diseases is often carried out on a trio (proband, mother and father). Here we will analyse a trio VCF of SNPs and INDELs to try and find the causal variant for the following case:

##### Clinical and Family History
A two year-old female presented for a well child check. Her early pediatric milestones were reassuring, she walked shortly after her first birthday and began talking around the same time. However, at the appointment, her mother notes that she is no longer using words that she used to and that she no longer feeds herself and seems less engaged. On examination, her growth parameters are in the normal range, but are notable for minimal growth in head circumference over the last 9 months. She verbalizes but does not make intelligible words and does not reciprocate in verbal exchange. She is an only child and the family history is negative with respect to significant medical problems.

###### HPO Terms
[Seizure](https://hpo.jax.org/app/browse/term/HP:0001250)  
[Spasticity](https://hpo.jax.org/app/browse/term/HP:0001257)  
[Postnatal microcephaly](https://hpo.jax.org/app/browse/term/HP:0005484)  
[Intellectual disability](https://hpo.jax.org/app/browse/term/HP:0002187)  

##### Trio Confirmation
When analysing a trio an important quality check is to confirm that the samples fit the expected relatedness. The relatedness between samples are specified in a [PED](https://www.cog-genomics.org/plink2/formats#fam) file called case3.ped. Use the program peddy to perform a quality control on the trio VCF. The peddy results are written in a html report file called **peddy_results.html**. More information on the peddy output can be found in the peddy [documentation](https://peddy.readthedocs.io/en/latest/).

All files needed are in the Case3 folder and the commands below assume that you are in the case3 folder.

```bash
    time singularity  exec --no-home -B /beegfs-storage/,$PWD:$PWD \
    docker://hydragenetics/peddy:0.4.8 \
    python -m peddy --procs $(nproc) \
    --prefix peddy_results \
    case3.vcf.gz case3.ped

```

    !!! questions
    :question: 
        1. Check the peddy report file. Do the samples show the expected relatedness and correct sex?
        2. How does peddy check the sex of the sample?

##### Using HPO terms  
We are also provided with HPO terms in the file case3_hpo.txt. We will use these HPO terms to try and identify some candidate genes. To do this we will use a tool that takes HPO terms and returns a prioritized list of genes associated with the phenotypes described by the HPO terms. Go to [https://phen2gene.wglab.org/](https://phen2gene.wglab.org/), paste in the hpo terms listed in the **hpo_terms.txt** in the box for HPO IDs and click submit.

Take the Gene that receives the highest ranking in the list and extract any variants in the VCF file from the gene. The trio VCF file has been annotated with gene information and population allele frequency data using VEP, so we can use bcftools or grep to extract the variants from the gene of interest.

    !!! questions
    :question: 
        1. What kind of pathogenic variant is present in this gene and what is its consequence? 
        2. What is the inheritance mode of the variant in this case?
        3. What phenotype on OMIM might fit with this inheritance mode and clincical history(hint: what phenotype has the same inheritance as our case and has similar phenotype)?

#### Case4

Here we will examine a VCF with structural variants from a patient 

##### Clinical and Family History
A patient diagnosed with colon cancer and with a family history of colon cancer. Genetic testing to determine if the patient has Lynch Syndrome (hereditary nonpolyposis colorectal cancer). The analysis of SNPs and Indel variants did not reveal any likely pathogenc variants. Therefore structural variant calling was performed using the structural variant Manta. Take a look inside the file called ```case4_manta.vep_annotated.vcf.gz```. You can see that this file is annotated with VEP and has information from ClinVar. We perform three filtration steps to narrow down the list of SVs: 

    1. Retain only records with a PASS in the filter field.
    2. Exclude variants with a frequency in Gnomad SV 4.0 > 0.05.
    3. Look for variants in a set of candidate genes.

Here we will use ClinVar's list of genes associated with Lynch Syndrome. We will then use the gene list to narrow down the list of variants in our VCF file. 

```bash

wget -qO - ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/gene_condition_source_id     | cut -f 2,5 | grep "Lynch syndrome" | cut -f1 | sort | uniq > lynch_syndrome_genes.txt

```

Use grep to extract variants that overlap with the genes listed above and view any detected regions in IGV.

    !!! questions
        :question: 
        1. Which genes are impacted by structural variants?
        2. Are the coding regions affected?
        3. What type of structural variants are they?


#### Case5
##### Repeat expansion detection

Here we will take a look at the difficult problem of calling expansions of short tandem repeats (STRs) with whole genome Illumina paired-end data.

##### Clinical and Family History
This male patient aged 40 has symptoms that include mild muscle weakness, myotonia, and cataracts. This patient is suspected to have Myotonic dystrophy type 1 (DM1).
Myotonic dystrophy type 1 is a progressive neuromuscular disease that occurs due to a CTG expansion in the 3' UTR of the DMPK gene. To detect a repeat expansion in DMPK we will use a program called ExpansionHunter. A description and explanation of the VCF produced by Expansion Hunter is available [here](
https://github.com/Illumina/ExpansionHunter/blob/master/docs/06_OutputVcfFiles.md). The BAM file used for this test contains reads mapped to a region of chr19 that contains the DMPK gene.

All files needed are in the Case5 folder and the commands below assume that you are in the case5 folder.

```bash
singularity  exec --no-home -B /beegfs-storage/,$PWD:$PWD \
    docker://hydragenetics/expansionhunter:5.0.0 \
    ExpansionHunter --reads case5.bam \
    --reference homo_sapiens.fasta  \
    --threads $(nproc) \
    --variant-catalog ${catalog} \
    --output-prefix case5.expansion_hunter
```


To visualise the read support for the expansion hunter results we will use a program calld REViewer which works with the small BAM file produced by expansionHunter. More information on REViewer is available in this [blog post]() from illumina and in the [REViewer publication](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-022-01085-z). 

```bash
singularity  exec --no-home -B /beegfs-storage/,$PWD:$PWD \
    docker://hydragenetics/reviewer:0.2.7 \
    REViewer --reads case5.expansion_hunter_realigned.sorted.bam \
    --vcf case5.expansion_hunter.vcf \
    --reference homo_sapiens.fasta \
    --catalog  variant_catalog_hg38.json \
    --locus DMPK \
    --output-prefix case5.reviewer

```

The image output by REViewer is in SVG format. This is best viewed in a web browser such as firefox or Google Chrome. Examine the reviewer plot to understand the division of the supporting reads into the spanning, flanking and in-repeat categories.

    !!! questions  
        :question: 
        1. Expansion Hunter takes a catalog of loci to genotype. How many loci in the variant_catalog_hg38.json file?
        2. Expansion hunter divides supporting reads into spanning, flanking and in-repeat
        reads. In this case which type of read offers the most support to the expanded allele call? 


We will then use the tool stranger to annotate the short tandem repeats in the ExpansionHunter VCF. This program adds additional information whether the length of the alleles estimated are pathological. See [here](https://github.com/Clinical-Genomics/stranger?tab=readme-ov-file#output) for further information on the stranger annotated vcf.

```bash
singularity  exec --no-home -B /beegfs-storage/,$PWD:$PWD \
    docker://hydragenetics/stranger:0.8.1 \
    stranger -f variant_catalog_hg38.json case5.expansion_hunter.vcf > case5.stranger_annotated.vcf

```

Use the stranger annotated VCF file to answer the following questions:

    !!! questions  
        :question: 
        1. How many repeats does the patient have in the longer allele and is it pathogenic?
        2. Is the shorter allele above the pathogenic threshold?
        2. The tri-nucleotide repeat motif for DMPK in the literature is given as CTG. Why does Expansion Hunter report a repeat motif of CAG?