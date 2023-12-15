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
* []()
* [CLINVAR]


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
    * Right click in the read track and choose Collapsed or Squished to see more of the data
    * Colour by Insert size and pair-orientation

### 2. Cases

#### Case1

TP53 mutation

#### Case 5


##### Clinical and Family History

A 55 year old male patient has been experiencing symptoms of lethargy and aches in his joints. Blood tests peformed displayed elevated levels of ferritin. The patient does not have a family history of confirmed hemochromotosis type 1. 

There are a number of ways to obtain a list of gene disease associations. In this case we have a good idea what the disease is and so we will search the Online Mendelian Inheritance in Man catalog for hemochromotosis type 1 and navigate to the molecular genetics section.

!!! questions
    :question: 
    In which gene should we be searching for variants?

Using bcftools or grep extract the variants for the relevant gene and identify any pathogenic mutations. 

    !!! questions
    :question: 
    What is the consequence of the variant?
    What is the impact of the variant?
    What genotype does the sample have and what is the inheritance mode of hemochromotosis type 1?

The CLINSIG field of the VEP annotation can contain multiple entries. Use the variant rs-id to search in CLINVAR and find more information on the molecular consequences of this variant.

    !!! questions
    :question: 
    What molecular consequences are listed for this variant? Is this different from those reported by VEP? Which transcript is reported in VEP and why?

#### Case3

Analysis or rare genetic diseases if often carries out on a trio (proband, mother and father). Here we will analyse a trio VCF of SNPs and INDELs to try and find the causal variant for the following case:

##### Clinical and Family History
A two year-old female presented for a well child check. Her early pediatric milestones were reassuring, she walked shortly after her first birthday and began talking around the same time. However, at the appointment, her mother notes that she is no longer using words that she used to and that she no longer feeds herself and seems less engaged. On examination, her growth parameters are in the normal range, but are notable for minimal growth in head circumference over the last 9 months. She verbalizes but does not make intelligible words and does not reciprocate in verbal exchange. She is an only child and the family history is negative with respect to significant medical problems.

##### HPO Terms
Seizure()
Spasticity()
Postnatal microcephaly()
Intellectual disability profound()

When analysing a trio an important quality check is to confirm that the samples fit the expected relatedness. The relatedness between samples are specified in a [PED](https://www.cog-genomics.org/plink2/formats#fam) file called case3.ped. Use the program peddy to perform a quality control on the trio VCF. The peddy results are written in a html report file called **peddy_results.html**. More information on the peddy output can be found in the peddy [documentation](https://peddy.readthedocs.io/en/latest/).


    !!! questions
    :question: 
        1. Do the samples show the expected relatedness and correct sex?
        3. Peddy's sex check uses genotype calls on ChrX to determine sex, can you suggest an alternative method using a BAM file with mapped reads?
        
Now that
The VCF file has been annotated with gene information and population allele frequency data using VEP. We are also provided with HPO terms in the file case3_hpo.txt. We will use these HPO terms to try and identify some candidate genes. To do this we will use a tool that takes HPO terms and returns a prioritized list of genes associated with the phenotypes described by the HPO terms.


#### Case4




#### Case5