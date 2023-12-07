<!-- ---
status: new
--- -->
# Computer lab 2: Sequencing quality control

This part of the lab picks up as the sequencing machine has finished. Even if we got the machine to run there is no guarantee that the data is actually what we expect and the quality is good enough to use in our downstream analysis.  
<br>
As the machine finishes it usually outputs some QC report, for Illumina machines this is in the Sequencing Analysis Viewer (SAV) but could also be a text file or similar. This QC-report usually contains information on how the sequencing went, i.e. how much data were produced, clustering and average basequality etc. If several samples were pooled there is also information on how much data each index(-pair) got, and how much of the data actually contains the specified indexes.
 <!-- screenshot of SAV? -->

## Prerequisites
* IGV
* Web browser
* Singularity
* Shell?

## Files
<!-- To be done! 
- fastqscreen
- multiqc
- vcf
    variant not reported in vcf
    variant reported in vcf
- bam
    - amplicon
    - capture
    - wgs
-->
## Common Quality Values
While running a bioinformatic pipeline it is important to continuously collect QC-values, this ensures that each step work as expected and can help you identify where and what happened if something gives unexpected results or crashes.  
<br>
![qcvalues dag](includes/images/qc_values.png){: width:150%"}  
<br>

One issue or hurdle is that the same word can be used for different quality-values, it can also be the same value but calculated at different timepoints or the algorithm might differ depending on which program you use. For example duplication rate is a value of how many copies of the same read your library has, this can be estimated optically from the fastq-files or calculated after the reads have been aligned to your reference genome. Even if you calculate the duplication rate on the same file the algorithm can also differ between different programs (or even version of program). For example, how you define a duplicated read can differ, do you look at just the starting point or so you look at both the starting and end position of the reads. Therefor it is **always** important to know that how and on what file a value is calculated.  
<br>

Another important factor to keep in mind is what kind of library you are looking at, capture, amplicon, pcr-free or even long-read all differs in expected values and results, and the values are often not comparable across methods.
??? abstract "Sequencing methods"
    **Short Read Sequencing**: most common sequencing method, read length ~50-150 bp.  
    **Long Read Sequencing**: allows for detection of complex SVs, read length ~5000-30000 bp.
    
    **PCR-free**: sequencing library created without the use of PCR.  
    **(Hybridization) Capture**: target sequencing using baits to "fish out" regions of interest.  
    **Amplicon**: target sequencing using primers to amplify specific targets.  

<br>
<!-- Samma som i lab1? skippa? -->
!!! question "Question 1"
    :question:  
    Open the bamfiles A, B and C in IGV. Can you identify what type of sample each is?

### Number of reads
Number of reads are what they sound like, how many reads do you have in your sample. Often this variable is used to for checking if the sequence machine and pooling behaved as expected. This number is also often used to calculate pooling, or expected output if you change flowcell/sequencing machine size.  

Number of reads are often calculated either directly in the sequencing machine, raw fastq or even the aligned bamfiles.  
<br>
*If you get a lower value for a sample than expected what error could that indicate?*  

Example of programs that calculate Number of reads:

* SAV/sequence machine run stats
* [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
* [Samtools stats](http://www.htslib.org/doc/samtools-stats.html)
* [Picard HSMetrics](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics)

!!! question "Question 2"
    Explain why the parameter "Number of Reads" differ between different programs for this Illumina Nextseq run of 16 samples.

    |  | SAV | FastQC | Samtools stats | Picard HSMetrics | 
    | --- | --- | --- | ---- | --- |
    | **Sample** | **`Total reads`** | **`Total Sequences`** | **`raw total sequences`** | **`TOTAL_READS`** | 
    | SampleA | <br /> 160M {: rowspan=2} | 5M | 8M | 10M |
    | SampleB | 5.3M | 9.2M | 11M |

    ??? tip
        FastQC is run with the command: `fastqc --quiet --outdir qc/fastqc/ fastq/sampleA_R1_001.fastq.gz`  
        Samtools stats: `samtools stats -t design_file.bed alignment/sampleA.bam  > qc/samtools_stats/sampleA.samtools_stats.txt`  
        Picard HsMetrics: `gatk CollectHsMetrics -COVERAGE_CAP 500 -BAIT_INTERVALS design_file.intervals -TARGET_INTERVALS design_file.intervals -INPUT alignment/sampleA.bam -OUTPUT qc/picard_collect_hs_metrics/sampleA.HsMetrics.txt`  
<!-- 
Svar:
fastqc 9 389 272 endast R1
samtools 10 272 580 med bedfil
hsmetrics 18 778 544  hela bam 
SAV hela korning 100M-1.2B 
-->

### Percent [%] mapped

After trimming the reads are aligned to a reference genome, the percent mapped is often tracked during this step. This is to measure how well your data fit the reference provided. 

<!-- Fixa fastq screen resultat med kontaminerat prov -->
!!! question "Question 3"
    Explain what might have happened here:  

    ![fastq_screen](includes/images/fastq_screen.png)
    Human contaminated with virus or virus contaminated with human
    

### Coverage
A measurement of how well covered your sample is. Usually defined as *"Y" x coverage*; which means the average position is covered by *Y* number of reads. There are many different programs that calculate coverage (often in combination with other QC parameters), some examples are:

- [Samtools Stats](http://www.htslib.org/doc/samtools-stats.html)
- [Picard CollectAlignmentSummaryMetrics](https://gatk.broadinstitute.org/hc/en-us/articles/13832683660955-CollectAlignmentSummaryMetrics-Picard-)
- [Mosdepth](https://github.com/brentp/mosdepth)

!!! question "Question 4"
    :question:  
    Calculate the average coverage of SampleA in the region chrX:1-100 with mosdepth.  
    `singularity exec docker://hydragenetics/mosdepth:0.3.2 mosdepth sampleA-output sampleA.bam`  
    ??? tip
        Looking into [mosdepth documentation](https://github.com/brentp/mosdepth), is there a parameter that can help you restrict you calculation to only the region of interest?
        
        ??? tip 
            How would you translate the region into a bedfile format?  
            `chrA    X   Y`

#### Coverage distribution and Fold80
Coverage distribution describes how the aligned reads are divided over the reference genome are aligned; Are they isolated to certain areas? Are some areas not covered properly? Are all regions covered the same?  
Just like you did in [lab 1](lab1.md), you can identify sequencing method by just looking at the distribution i.e. in the coverage track in IGV. Coverage distribution is also used to estimate CNVs by most CNV-algorithms.  

![mosdepth_percontig](includes/images/mosdepth-coverage-per-contig.png)

!!! question "Question 5"
    Looking at the coverage distribution (above) for 16 samples (same capture panel, same sequence run), what stands out? 
    ??? tip
        Are there a difference between autosomal and sex chromosomes?  
        ??? tip "Autosomal chromosomes"
            Looking at the autosomal chromosome there is one sample that stands out. This is its CNV profile. What does that tell us?  
            ![cnv_profile](includes/images/cnv_profile_12add.png)
        ??? tip "Sex chromosomes"
            How many copies do we have of the sex chromosomes compared to autosomal, and how does this correlate to coverage?  

<br>
A more quantitative value based on coverage distribution is the fold 80 base penalty value calculated by e.g. [Picard CollectHsMetrics](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics). The **Fold80** is defined as the fold change of non-zero read coverage needed to bring 80% of the ROI bases to the observed mean coverage, or a bit simpler put: how much more sequencing is required to bring 80% of the target bases to the mean coverage.

#### Coverage breadth
Yet another common measurement of coverage is coverage breadth, or % >= Yx as it is often described as. This means that X % of the genome or target regions is covered by **at least** Y reads. By default several of these values are calculated using the different [Picard](http://broadinstitute.github.io/picard/picard-metric-definitions.html) programs depending one your sequencing (wgs, capture amplicon etc). But most coverage programs allows you to specify your own thresholds if you want specific level(s). Since this is a measurement of coverage of a given **target** it is important to keep track of what region/target this is calculated over to be able to compare numbers.

### Duplication rate
The duplication rate is the fraction of mapped reads marked as duplicate reads in a particular data set. In contrast to overlapping reads, duplicate reads offer no additional information and are removed from sequencing data during the bioinformatic pipeline, a process known as deduplication. With other words  duplicate reads are identical reads which are an artefact/bias introduced by a method, biological or technical (usually but not exclusively PCR), and will not give any new information and should therefore be flagged and remove from your dataset.  

As most QC-values duplication rate can be calculated at many different stages and the definition can vary depending on algorithm.

The most common program for calculating the duplication rate on **unaligned reads** is [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). FastQC can either be a part of the bioinformatic pipelines or on some newer sequencing machines it is included in the "SAV".

For **aligned reads**, [Picard MarkDuplicates](https://gatk.broadinstitute.org/hc/en-us/articles/360037052812-MarkDuplicates-Picard-) is the most common program that identifies, flags and keeps outputs statistics of duplicates reads, but there are plenty of other programs and in-house solutions that does the same thing. 
<!-- 
#### Unaligned
Fastqc 100 000 reads 50 bp identical
#### Aligned
picard, mapped 5' end of read -->

!!! question "Question 6"
    :question:
    Find the estimated duplication rate for sampleA from both FastQC and Picard MarkDuplicates.
    ??? tip
        Checkout the output from the different programs under the `qc`-folder.
    :question:  
    Explain why the duplication rate differs between FastQC and Picard MarkDuplicates?
    ??? tip
        Check-out [FastQC's](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) documentation and compare it with [Picard Markduplicate's](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics).

---

| Sample  | Number of reads | Duplication rate | Median Coverage | % > 30x | > 25x |  Fold80 | % Adapter trimmed | 
| --- | --- | --- | ---- | --- | --- | --- | --- |
| SampleA | | FastQC: MarkDuplicated: |  |  | | | |
| SampleB | | FastQC:  MarkDuplicated: |  |  | | | |

!!! question "Question 7"
    Fill out the table above with the missing values for sampleA.

    ??? tip
        Navigate the different outputs under the `qc`-folder to find the missing values.
        ??? tip "Percent Adapter Trimmed"
            What step would adapter trimming happen in? Do you do it before or after aligning you reads to the reference genome? Is there another folder matching that step, could you find any log files there?
        

## MultiQC
As you might have notice it takes quite a while to identify and find the different QC-values to track. To combat the never-ending file searching and enable easier tracking of QC-values the program [MultiQC](https://multiqc.info/) is often used. MultiQC aggregate the different QC-values into a single report. MultiQC only aggregate results from other programs and never does any calculation, therefor what values you can find and the layout of a MultiQC report varies a lot. It also allows you to configure almost all parts of the report with a config so a MultiQC-report does not always look like a MultiQC-report.  

<br>
The report is a self-contained interactive `.html`-file where you can adapt/configure/highlight etc the plots and tables directly in your web-browser. It also allows for an easy export of table and plots if needed. 

!!! question "Question 8"
    Fill in the same table as in Question 7 but for SampleB with the values found in the MultiQC report.

!!! question "Question 9"
    Try generating a new MultiQC and add XYZ modules.
    ??? tip
        Check the `multiqc_config.yaml` if there is something in there that can be changed?
<!-- 
extra assignments/question 9
edit a ready multiqc report mha config, tex ordningen eller lägga till module baserat på vad för filer som finns preppade?
-->

## Variant Quality
The next step after you have validated the quality of the sequencerun is variants. How do we know that a variant we see is real and not and not an artefact?

### BaseQ and MapQ
There are several different ways and steps to validate a variant. Most variant callers have build-in filtering steps that ensures that the variants reported are filtered or at least flagged if the confidence is low. One of the easiest way is to look at the [base quality](lab1.md)<!--find specific passage about phredsscore --> of the variant's alternative bases. Of course does baseQ vary depending sequencing [machine and/or version of said machine](https://en.wikipedia.org/wiki/FASTQ_format#Encoding). A good rule-of-thumb is that you always want at least a phred-score of at least 20 to be able to trust a base-call.

The other quality that is usually included are mapping quality, how well a read has aligned to exactly that specific region of the reference genome. Different aligners use different scoring algorithm and there for it is hard to compare quality values across programs. For some more in-depth reading see [here](http://www.acgt.me/blog/2014/12/16/understanding-mapq-scores-in-sam-files-does-37-42) and [here](https://davetang.org/muse/2011/09/14/mapping-qualities/) for example.

!!! question "Question 10"
    Open `.bam` file for sampleA in IGV and navigate to X. Why do you think this variant was not reported in the vcf from the variantcalling. Does something differ from Y, why was that reported in the vcf?
    ??? tip
        Sort the position based on base (right click -> sort by -> base). Can you see something in when looking at the base-calls?

### Genotype Quality
Most variant callers have some sort of genotype quality scoring available for each variantcall. The problem here is that it is not standardized and each caller can have completely different values making them hard to compare. For example for the GATK produced vcf-files there are several different quality values, there is the `QUAL` column as well as the `GQ` and `PL` fields in the FORMAT column. For more in-depth info on the different vcf columns and annotation for GATK see more info [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035531692-VCF-Variant-Call-Format).

![gq_values](includes/images/gq_values.png)
<!-- Nice plot of GQ values for deepvariant vs haplotype -->

### Background and Artefacts
<!-- Something about how to calc AF? -->