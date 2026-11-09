# Computer lab 3 - Germline and Somatic Variants
In this computer lab we are going to explore different kinds of germline and somatic variants.

## Overview
1. Visualise germline structural variants and somatic variants in IGV
2. Find and examine potentially pathogenic germline variants:
    - Case 1: Pathogenic SNV in TP53 gene
    - Case 2: Structural variant underlying a heritable cancer syndrome
3. Somatic copy number variations
4. Repeat expansion detection (**Optional** extra case)

## Prerequisites
All of the below are installed or available through the module system on Pelle

* [IGV](https://igv.org/) - Desktop application or web app
* Shell / terminal / putty
* rsync
* singularity
* bcftools

## Interactive jobs on Uppmax

In this lab all analysis will be performed in an interactive job. The following command can be used to launch an interactive session for 4 hours with 1 core allocated.

```bash
interactive -A uppmax2024-2-1 -t 4:00:00
```

## Files
Login to [pelle](https://docs.uppmax.uu.se/cluster_guides/pelle/) on UPPMAX.
Copy the lab files to your home directory on pelle (UPPMAX).

```bash
cp -r /proj/uppmax2024-2-1/uppmax2024-2-1/nobackup/lab_files/lab3_germline_and_somatic/ ~/
cd lab3_germline_and_somatic
```

It is also recommended to download the files to your local computer using scp, [FileZilla](https://uppsala.instructure.com/courses/115337/pages/things-to-do-before-the-course-starts) or similar (for easier viewing of bam and html files). Example using scp on your local machine (exchange {username} with your own username on uppmax):

```bash
scp -r {username}@pelle.uppmax.uu.se:~/lab3_germline_and_somatic .
```

## Links to resources
* [Genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/)
* [IGV desktop Documentation](https://igv.org/doc/desktop/)
* [VEP glossary](https://www.ensembl.org/info/website/glossary.html)
* [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/)
* [CNV caller: CNVkit](https://cnvkit.readthedocs.io/en/stable/pipeline.html)
* [STRipy STR database](https://stripy.org/database)

## 1. Visualising Variants in IGV
The visual inspection of the sequencing data supporting a variant is an important part of a clinical bioinformaticians work. For additional information regarding genetic variant types see [genetic variants explained](https://bitesizebio.com/23996/whats-so-important-about-variants/).

### Germline structural variants
The reads in this BAM file have been mapped to the **hg38** reference genome, so make sure to switch to this genome version in IGV.

Open IGV and then open the `NA12878_examples.bam` file in the `sv_examples` folder. In the regions below there are different types of structural variants. For additional information on interpreting read mapping in IGV see the [IGV documentation](https://igv.org/doc/desktop/#UserGuide/tracks/alignments/paired_end_alignments/#insert-size) on paired end alignments.

**Regions:**

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
    * In IGV click View > Preferences > Alignment > Show soft-clipped bases
    * Copy the regions into IGV
    * Zoom in or out as needed
    * View as pairs
    * Color alignments by -> insert size and pair orientation
    * Sort alignments by -> insert size

### Somatic variants
Make a new session in IGV and open the `HD832_T.variant.genes.bam` file. This BAM file is mapped to **Human hg19**, so switch the reference genome.

Somatic variants with low allele frequency do not show up in the coverage track. To mitigate this **right-click on the coverage track, then set allele frequency threshold to 0.05.**

**In Questions 2 - 5 we use the same regions so it might be convenient to answer all these questions at once for each region.**

**Regions:**

* chr7:140453022-140453248
* chr2:29416330-29416416
* chr7:140434558-140434634
* chr3:41265991-41266273
* chr2:29416005-29416044

!!! TIP
    * Left click on positions in the coverage track to see support for each allele
    * Right click in the read track and choose Collapsed or Squished to see more of the data

!!! question "Question 2"
    :question:
    In the regions above there are variants, determine which variant is of which type (there can be more than one). **Only variants with allele frequency above 5%** should be considered as this is the limit used in the clinic for this sequencing panel.

    * Region with many artifacts
    * Somatic SNV
    * Germline SNV
    * Deletion
    * Insertion

!!! question "Question 3"
    :question:
    **In questions 3-5, ignore the artifact region.** Determine the exact genomic starting position of all non-artifact variants

!!! question "Question 4"
    :question:
    Determine the number of supporting reads for all non-artifact variants

!!! question "Question 5"
    :question:
    Determine the allele frequency for all non-artifact variants. **Allele frequency for deletions and insertions needs manual calculation**
    ??? note
        IGV uses all reads available in the bamfile, no filtering except duplicate reads is applied by default. Therefore the calculated "raw" allele frequency from IGV does not always correlate with the AF found in vcf-files since callers perform additional filtering of reads before calculating AF.

---
## 2. Germline Cases

### Case 1 - Pathogenic SNV in TP53 gene

#### VCF with VEP annotations
Here we will take an example VCF file with a pathogenic variant in the TP53 gene. This case will serve as an example on how to filter variants in a vcf file and how to extract information about the variants. **There are no questions for this case**.

Here you are given a VCF file called `case1.vep_annotated.vcf.gz` containing SNVs and INDELs. The VCF has been annotated with variant effect information using the [Variant Effect Predictor](https://www.ensembl.org/info/docs/tools/vep/index.html) tool to calculate the [variant consequences](https://www.ensembl.org/info/genome/variation/prediction/predicted_data.html) of each variant in the VCF. The variant information added by VEP is located in the INFO field starting with "CSQ=".

We can extract information about what the various fields in the VEP annotations are using a grep search of the VCF header:

```bash
zgrep ^# case1.vep_annotated.vcf.gz | grep CSQ
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
module load BCFtools/1.22
```

Once loaded we can list all the available VEP subfields from the CSQ INFO field

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

Using bcftools also allows the output to be formatted in a more human readable way where a subset of columns that are of most interest can be extracted (e.g., Consequence and CLIN_SIG can be extracted to look for evidence of pathogenicity):
```bash
bcftools view -f PASS case1.vep_annotated.vcf.gz |  bcftools +split-vep -c MAX_AF:Float,SYMBOL -e "MAX_AF>0.1 | SYMBOL!='TP53'" -f "%CHROM %POS %SYMBOL %Consequence %IMPACT %Protein_position %Amino_acids %CLIN_SIG [%GT ]\n"
```

### Case 2 - Structural variant underlying a heritable cancer syndrome
The files for this case are in the `case2` folder.

#### Clinical and Family History
A patient diagnosed with colon cancer and with a family history of colon cancer. Genetic testing is performed to determine if the patient has Lynch Syndrome (hereditary nonpolyposis colorectal cancer). The analysis of SNVs and INDEL variants did not reveal any likely pathogenic variants. Therefore structural variant calling was performed using the structural variant (SV) caller [Manta](https://github.com/Illumina/manta) which combines paired and split-read evidence to call SVs.

#### Variant filtration
Take a look inside the file called `case2_manta.vep_annotated.vcf.gz` using `less -S`. You can see that this file is annotated with VEP and has information from ClinVar and Genome Aggregation Database (gnomAD) SV v4.0. We will perform three filtration steps to narrow down the list of SVs:

1. Retain only records with a PASS in the filter field.
2. Look for variants that overlap with a set of candidate genes in the `lynch_syndrome_genes.bed`. The genes in this file were obtained from ClinVar's list of genes associated with Lynch Syndrome.
3. Exclude variants with an allele frequency in gnomAD SV 4.0 > 0.1 (called gnomad_AF in the VCF VEP annotation field).

!!! note
    For step 2 you can use the `-R lynch_syndrome_genes.bed` with `bcftools view` command to extract variants that overlap with the genes listed in the bed file.

Use the filtered vcf generated from Steps 1-3 above and the `case2_sv.bam` file to answer the following questions:

!!! question "Question 6"
    :question:
    Which of the Lynch Syndrome genes are affected by a structural variant?

!!! question "Question 7"
    :question:
    What type of structural variant affects this gene and how large is it?

!!! question "Question 8"
    :question:
    Using the `case2_sv.bam` file, view the structural variant in IGV. Do the patterns of insert size and read coverage support for the structural variant indicate that the call made by Manta is correct?
    ??? tip "Hint"
        View as pairs option in IGV can help visualise the effect of the SV on insert size

---
## 3. Somatic Copy Number Variation (CNV)

In cancer cells large and numerous copy number alterations are quite common, especially in certain solid tumors like lung cancers. CNVs are called by cancer specific callers like [CNVkit](https://cnvkit.readthedocs.io/en/stable/pipeline.html).

Open `CNV/tumor.cnv.html` in a browser. The first plot shows the zoomed-in view of the data and the second plot an overview of the entire genome. There is also a table to the right with potentially clinically relevant CNVs.

!!! TIP
    * Click on a chromosome in the overview to show it in the top plot.
    * In the top plot, left-click and drag to zoom in.
    * In the top plot, left-click to zoom out to chromosome view.

!!! NOTE
    Log2ratio is the copy number in the sample and is calculated as a ratio to the expected number of copies (diploid genome = 2) and then $ \log_2 $ is applied. A normal copy number of 2 is therefore $ \log_2(\frac {2}{2})=\log_2(1)=0 $.

    If the sample instead has an extra copy then $ \log_2(\frac {3}{2})=\log_2(1,5)=0.6 $

    Or one lost copy is $ \log_2(\frac {1}{2})=\log_2(0,5)=-1 $.

    However, this only applies if the tumor content is 100%. Otherwise, the effect of a somatic CNV will be smaller as only a subset of the cells are affected. See further [CNVkits explanation of log2ratio](https://cnvkit.readthedocs.io/en/stable/calling.html).

!!! NOTE
    The VAF-plots show the allele frequencies of heterozygous **germline** SNPs in the sample. When there is a somatic deletion or duplication the allele frequencies will increase or decrease depending on the allele. This is used as additional evidence of amplifications and deletions.

!!! question "Question 9"
    :question:
    Based on the Genome view, which chromosomes have large deletions? (Always take special care when looking at the X chromosome)

!!! question "Question 10"
    :question:
    Based on the Genome view, which chromosomes have large duplications?

### Optional - Clinically relevant CNVs

#### Sample with clinically relevant amplification
In this sample (`CNV/EGFR_amp.cnv.html`) there is an amplification of EGFR. Cancers with this amplification are often resistant to tyrosine kinase inhibitors and can therefore affect the treatment of the patient.

Find the EGFR amplification and zoom in on the gene.

!!! TIP
    **When there are really high amplifications the data is sometimes outside the default scale. Click the box "Zoom to data extent" to see all the data.**

!!! question "Question 11"
    :question:
    What is the approximate log2 ratio, based on the plot, of the EGFR amplification? **Do not miss the tip above!**

!!! question "Question 12"
    :question:
    The table reports the copy number of the EGFR amplification after taking tumor content into account. What is the actual copy number found in the tumor based on the table?

#### Sample with clinically relevant deletion
In the sample `CNV/CDKNA_B_del.cnv.html` there is a homozygous deletion of the genes CDKN2A and CDKN2B. This is a diagnostic marker in some cancer types, associated with poor prognosis.

Find the CDKN2A and CDKN2B deletion.

!!! question "Question 13"
    :question:
    Based on the copy number plot and the VAF-plot it seems that most of chromosome 9 has only one copy in the tumor (heterozygous deletion). Motivate, based on both the log2 ratio and the VAF plot, why the region around CDKN2A and CDKN2B is a homozygous deletion in the tumor (both alleles deleted).

---
## 4. Optional - Repeat expansion detection

Next we will take a look at the difficult problem of calling expansions of short tandem repeats (STRs) with whole genome Illumina paired-end data.

#### Clinical and Family History
This male patient, age 40, has symptoms that include mild muscle weakness, myotonia, and cataracts. This patient is suspected to have Myotonic dystrophy type 1 (DM1). Myotonic dystrophy type 1 is a progressive neuromuscular disease that occurs due to a CTG expansion in the 3' UTR of the DMPK gene.

#### Repeat expansion genotyping
To detect a repeat expansion in DMPK we will use a program called ExpansionHunter. A description and explanation of the VCF produced by ExpansionHunter is available [here](https://github.com/Illumina/ExpansionHunter/blob/master/docs/06_OutputVcfFiles.md). The `case3.bam` file contains reads mapped to a region of chr19 that contains the DMPK gene.

All files needed are in the `case3` folder and the commands below assume that you are in that folder.

```bash
singularity exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/uppmax2024-2-1/nobackup/singularity_images_lab3/expansionhunter_5.0.0.sif \
    ExpansionHunter --reads case3.bam \
    --reference homo_sapiens.fasta  \
    --variant-catalog variant_catalog_hg38.json  \
    --output-prefix case3.expansion_hunter
```

Before we proceed further we need to sort and index the small bam file produced by ExpansionHunter using samtools available through the module system on Uppmax.

```bash
module load SAMtools/1.22

samtools sort -O BAM  case3.expansion_hunter_realigned.bam -o case3.expansion_hunter_realigned.sorted.bam
samtools index case3.expansion_hunter_realigned.sorted.bam
```

To visualise the read support for the ExpansionHunter results we will use a program called REViewer which works with the small BAM file produced by ExpansionHunter. More information on REViewer is available in this [blog post](https://www.illumina.com/science/genomics-research/articles/reviewer-alignments-short-reads-long-repeat.html) from Illumina and in the [REViewer publication](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-022-01085-z).
REViewer requires the sorted `case3.expansion_hunter_realigned.sorted.bam`, the `case3.expansion_hunter.vcf` output by ExpansionHunter and the variant catalog file to produce a plot for the DMPK locus.

```bash
singularity exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/uppmax2024-2-1/nobackup/singularity_images_lab3/reviewer_0.2.7.sif \
    REViewer --reads case3.expansion_hunter_realigned.sorted.bam \
    --vcf case3.expansion_hunter.vcf \
    --reference homo_sapiens.fasta \
    --catalog  variant_catalog_hg38.json \
    --locus DMPK \
    --output-prefix case3.reviewer
```

The image output by REViewer is in SVG format. This is best viewed in a web browser such as Firefox or Google Chrome. You can download
the file called `case3.reviewer.DMPK.svg` and open it with your web browser. Examine the REViewer plot to understand the division of the supporting reads into the spanning, flanking and in-repeat categories, and to assess the coverage for each allele.

We will now use the tool [Stranger](https://github.com/Clinical-Genomics/stranger) to annotate the short tandem repeats in the ExpansionHunter VCF. This program adds additional information whether the length of the alleles estimated by ExpansionHunter are pathological. See [here](https://github.com/Clinical-Genomics/stranger?tab=readme-ov-file#output) for further information on the stranger annotated vcf.

```bash
singularity exec --no-home -B $PWD:$PWD \
    /proj/uppmax2024-2-1/uppmax2024-2-1/nobackup/singularity_images_lab3/stranger_0.8.1.sif \
    stranger -f variant_catalog_hg38.json case3.expansion_hunter.vcf > case3.stranger_annotated.vcf
```

Use the `case3.stranger_annotated.vcf` file to answer the following questions:

!!! question  "Question 14"
    :question:
    How many repeats does the patient have in each allele?

!!! question  "Question 15"
    :question:
    Are both DMPK alleles above the pathogenic threshold?

!!! question  "Question 16"
    :question:
    ExpansionHunter divides supporting reads into spanning, flanking and in-repeat
    reads. How many spanning, flanking and in-repeat reads support the longer expanded allele?

A good resource for staying updated on pathogenic expansion repeats is the [STRipy STR database](https://stripy.org/database). Go to the DMPK entry to see information on this repeat expansion and the frequency of different repeat lengths in gnomAD.

!!! question  "Question 17"
    :question:
    The tri-nucleotide repeat motif for DMPK in the literature is given as CTG. Why does ExpansionHunter report a repeat motif of CAG?
