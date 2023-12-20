<!-- ---
status: new
--- -->
# Computer lab 2: Sequencing quality control

This part of the lab picks up as the sequencing machine has finished. Even if we got the machine to run there is no guarantee that the data is actually what we expect and the quality is good enough to use in our downstream analysis.  
<br>
As the machine finishes it usually outputs some QC report, for Illumina machines this is in the Sequencing Analysis Viewer (SAV) but could also be a text file or similar. This QC-report usually contains information on how the sequencing went, i.e. how much data were produced, clustering and average basequality etc. If several samples were pooled there is also information on how much data each index(-pair) got, and how much of the data actually contains the specified indexes.

## Prerequisites
* IGV
* Web browser
* Access to Uppmax project

If running locally:

* Singularity
* awk

## Files
Can be found at the Uppmax project.
Either download the entire folder locally or all work can be done in your own homefolder on Rackham.
To ensure that you don't accidently overwrite someone else file start by copying the entire `lab2` folder to your home (or local computer)
```bash
mkdir ~/${course_folder}
cp -r ${project_path}/lab2 ~/${course_folder}
cd ~/${course_folder}/lab2/
```
---

## Common Quality Values
While running a bioinformatic pipeline it is important to continuously collect QC-values, this ensures that each step work as expected and can help you identify where and what happened if something gives unexpected results or crashes.  
<br>
![qcvalues dag](includes/images/qc_values.png){: width:150%"}  
<br>

One issue or hurdle is that the same word can be used for different quality-values, it can also be the same value but calculated at different timepoints or the algorithm might differ depending on which program you use. For example duplication rate is a value of how many copies of the same read your library has, this can be estimated optically from the fastq-files or calculated after the reads have been aligned to your reference genome. Even if you calculate the duplication rate on the same file the algorithm can also differ between different programs (or even version of program). For example, how you define a duplicated read can differ, do you look at just the starting point or so you look at both the starting and end position of the reads. Therefor it is **always** important to know that how and on what file a value is calculated.  
<br>

Another important factor to keep in mind is what kind of library you are looking at, capture, amplicon, pcr-free or even long-read all differs in expected values and results, and the values are often not comparable across methods.
!!! abstract "Sequencing methods"
    **Short Read Sequencing**: most common sequencing method, read length ~50-150 bp.  
    **Long Read Sequencing**: allows for detection of complex SVs, read length ~5000-30000 bp.
    
    **PCR-free**: sequencing library created without the use of PCR.  
    **(Hybridization) Capture**: target sequencing using baits to "fish out" regions of interest.  
    **Amplicon**: target sequencing using primers to amplify specific targets.  

<br>

### Number of reads
Number of reads are what they sound like, how many reads do you have in your sample file(s). Often this variable is used to for checking if the sequence machine and pooling behaved as expected. This number is also used to calculate pooling, or expected output if you change flowcell/sequencing machine size.  

Number of reads are calculated either directly in the sequencing machine, raw fastq or even the aligned bamfiles.  

!!! question "Question 1"
    :question: If you get a lower number of reads for a sample than expected, what (error) could that indicate?  
<br>
Example of programs that calculate Number of Reads:

* SAV/sequence machine run stats
* [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
* [Samtools stats](http://www.htslib.org/doc/samtools-stats.html)
* [Picard HSMetrics](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics)

!!! question "Question 2"
    :question: Explain why the parameter "Number of Reads" differ between different programs for this Illumina Nextseq run of 16 capture samples.

    |  <br />**Sample** | SAV: <br />  **`Total reads`** | FastQC: <br /> **`Total Sequences`** | Samtools stats: <br /> **`raw total sequences`** | Picard HSMetrics: <br /> **`TOTAL_READS`** | 
    | --- | --- | --- | ---- | --- |
    | SampleA | <br /> 160M {: rowspan=2} | 5M | 8M | 10M |
    | SampleB | 5.3M | 9.2M | 11M |

    ??? tip
        Commands used to produce QC output:  
        **FastQC**: `fastqc --quiet --outdir qc/fastqc/ fastq/sampleA_R1_001.fastq.gz`  
        **Samtools stats**: `samtools stats -t design_file.bed alignment/sampleA.bam  > qc/samtools_stats/sampleA.samtools_stats.txt`  
        **Picard HsMetrics**: `gatk CollectHsMetrics -COVERAGE_CAP 500 -BAIT_INTERVALS design_file.intervals -TARGET_INTERVALS design_file.intervals -INPUT alignment/sampleA.bam -OUTPUT qc/picard_collect_hs_metrics/sampleA.HsMetrics.txt`  
<!-- 
Svar:
fastqc 9 389 272 endast R1
samtools 10 272 580 med bedfile
hsmetrics 18 778 544  hela bam 
SAV hela korning 100M-1.2B 
-->

### Percent [%] mapped

After trimming the reads are aligned to a reference genome, the percent mapped is often tracked during this step. This is to measure how well your data fit the reference provided. 

<!-- Fixa fastq screen resultat med kontaminerat prov 
module load bioinfo-tools
module load fastq_screen/0.15.2

fastq_screen --get_genomes

fastq_screen --conf $config $contaminated_fastq
fastq_screen --conf $config $ok_fastq
-->
!!! question "Question 3"
    :question: Explain what might have happened here:  

    ![fastq_screen](includes/images/fastq_screen.png)
    Human contaminated with mouse 
    

### Coverage
A measurement of how well covered your sample is. Usually defined as *"Y" x coverage*; which means the average position is covered by *Y* number of reads. There are many different programs that calculate coverage (often in combination with other QC parameters), some examples are:

- [Samtools Stats](http://www.htslib.org/doc/samtools-stats.html)
- [Picard CollectAlignmentSummaryMetrics](https://gatk.broadinstitute.org/hc/en-us/articles/13832683660955-CollectAlignmentSummaryMetrics-Picard-)
- [Mosdepth](https://github.com/brentp/mosdepth)

!!! question "Question 4"
    :question: Calculate the average coverage of TP53's exon3 (hg19: chr17:7579312-7579590) in SampleB with mosdepth.  
    Either you do this locally on your computer (need to have singularity available), or on Uppmax.  
    *(Bam-file aligned to Hg19 reference genome.)*  

    ```bash
    cd ${project_folder}/${user_folder}/
    module load bioinfo-tools
    module load mosdepth
    mosdepth sampleB-output ${project_folder}/lab2/sampleB_subset.bam
    ```

    or if using singularity locally you need download bamfile and index files from Uppmax first, and then run mosdepth locally using singularity
    ```bash
    # Files needed: sampleB_subset.bam and sampleB_subset.bam.bai
    singularity exec docker://hydragenetics/mosdepth:0.3.2 mosdepth sampleB-output sampleB_subset.bam
    ```
    
    ??? tip
        Looking into [mosdepth documentation](https://github.com/brentp/mosdepth) or using the built-in help `mosdepth --help`, is there a parameter that can help you restrict you calculation to only the region of interest?
        
        ??? tip 
            How would you translate the region into a bedfile format?  
            `chrA    X   Y`

#### Coverage distribution
Coverage distribution describes how the aligned reads are divided over the reference genome are aligned; Are they isolated to certain areas? Are some areas not covered properly? Are all regions covered the same?  
Just like you did in [lab 1](lab1.md#assignment-3), you can identify sequencing method by just looking at the distribution i.e. in the coverage track in IGV. In-depth coverage distribution is also used to estimate CNVs by most CNV-algorithms.  

![mosdepth_percontig](includes/images/mosdepth-coverage-per-contig.png)

!!! question "Question 5"
    :question: Looking at the coverage distribution (above) for 16 samples (same capture panel, same sequence run), what stands out? 
    ??? tip
        Are there a difference between autosomal and sex chromosomes?  
        ??? tip "Autosomal chromosomes"
            Looking at the autosomal chromosome there is one sample that stands out. This is its CNV profile. What does that tell us?  
            ![cnv_profile](includes/images/cnv_profile_12add.png)
        ??? tip "Sex chromosomes"
            How many copies do we have of the sex chromosomes compared to autosomal, and how does this correlate to coverage?  


#### Fold80
A more quantitative value based on coverage distribution is the fold 80 base penalty value calculated by e.g. [Picard CollectHsMetrics](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics). The **Fold80** is defined as the fold change of non-zero read coverage needed to bring 80% of the ROI bases to the observed mean coverage, or a bit simpler put: how much more sequencing is required to bring 80% of the target bases to the mean coverage.

#### Coverage breadth
Yet another common measurement of coverage is coverage breadth, or **% &ge; Yx** as it is often described as. This means that X % of the genome or target regions is covered by **at least** Y reads. By default several of these levels are calculated in most of the [Picard](http://broadinstitute.github.io/picard/picard-metric-definitions.html) metrics programs (depending one your sequencing method which collect metrics you run may vary). Since this is such a popular value many coverage programs often calculates breadth and allows you to specify your own specific thresholds if needed, but the most common program used many of the [Picard metrics](http://broadinstitute.github.io/picard/picard-metric-definitions.html) algorithms. Since this is a measurement of coverage over a given **target** it is important to keep track of what region/target this is calculated over to be able to compare numbers.

!!! question "Question 6"
    Why might the coverage breadth vary for the same human wgs (reference genome HG38) sample calculated with different pipelines?

    | Name | PipelineA: Median Cov | PipelineA: % &ge; 30x | PipelineB: Median Cov | PipelineB: % &ge; 30x |
    | --- | --- | ---| --- | ---|
    | SampleC | 34 | 91 | 34 | 99 | 
    ??? tip
        Is can we map reads to the entire genome?


### Duplication rate
The duplication rate is the fraction of mapped reads marked as duplicate reads in a particular data set. In contrast to overlapping reads, duplicate reads offer no additional information and are removed from sequencing data during the bioinformatic pipeline, a process known as deduplication. With other words  duplicate reads are identical reads which are an artefact/bias introduced by a method; biological or technical (usually but not exclusively PCR), and will not give any new information and should therefore be flagged and remove from your dataset.  

As most QC-values duplication rate can be calculated at many different stages and the definition can vary depending on algorithm.

The most common program for calculating the duplication rate on **unaligned reads** is [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). FastQC can either be a part of the bioinformatic pipelines or on some newer sequencing machines it is included in the "SAV".

For **aligned reads**, [Picard MarkDuplicates](https://gatk.broadinstitute.org/hc/en-us/articles/360037052812-MarkDuplicates-Picard-) is the most common program that identifies, flags and records outputs statistics of duplicates reads, but there are plenty of other programs and in-house solutions that does the same thing. Picard also has a program called [Collect Duplicate Metrics](https://gatk.broadinstitute.org/hc/en-us/articles/360057441491-CollectDuplicateMetrics-Picard-) which calculates the duplication rate on a previously duplication-marked bamfile.
<!-- 
#### Unaligned
Fastqc 100 000 reads 50 bp identical
#### Aligned
picard, mapped 5' end of read -->

!!! question "Question 7"
    :question: Find the estimated duplication rate for sampleA from both FastQC and Picard CollectDuplicateMetrics.
    ??? tip
        Checkout the output from the different programs under the `qc`-folder.
    :question: Explain why the duplication rate differs between FastQC and Picard MarkDuplicates/CollectDuplicateMetrics?
    ??? tip
        Check-out [FastQC's](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) documentation and compare it with [Picard Markduplicate's](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics).

---

| Sample  | Number of reads | Duplication rate | Mean Coverage | % &ge; 200x | % &ge; 500x |  Fold80 |
| --- | --- | --- | ---- | --- | --- | --- | 
| SampleD | | FastQC: <br />CollectDuplicateMetrics: |  |  | | |
| SampleE | | FastQC: <br />CollectDuplicateMetrics: |  |  | | |

!!! question "Question 8"
    :question: Fill out the table above with the missing values for sampleD.

    ??? tip
        Navigate the different outputs under the `qc`-folder to find the missing values for sampleD.
        ??? tip "Programs"
            Number of reads: fastqc  
            Duplication rate: fastqc or collect duplicate metrics  
            Mean coverage: mosdepth  
            Coverage breadth: mosdepth  
            Fold80: Picard HsMetrics  
    ??? tip "Tip: Coverage breadth"
        Finding coverage breadth is not always easy, in the mosdepth_bed folder, locate the `qc/mosdepth_bed/sampleD_T.thresholds.bed.gz` file and have a look inside it with `less ${file}` (use `q` to exit).  What you want is the total coverage breadth for 200x and 500x, not per region. Identify which columns show per region coverage breadth.  
        To calculate the % &ge; Yx you need to add together coverage for the entire design and then divide it by the total length of the design: 
        $$ \frac{\sum covered\ bases} {\sum region\ length} $$
        
        For example:

        |  |  |  | 10x|
        |---|---|---|---|
        |chrA |1 |11|2|
        |chrA|20|25|0|

        Would give us $ \frac {(2+0)} {(11-1)+(25-20)}=0.08 $ or 8 % &ge; 10x

        Identify which columns you are interested in and use it to replace `$x` with the column number (1-index) in the script below:
        ``` bash
        zcat qc/mosdepth_bed/sampleD_T.thresholds.bed.gz | awk 'NR>1{tot_length+=$3-$2;tot_bases+=$x}END{print tot_bases"/"tot_length"="tot_bases/tot_length}'
        ```bash
        Step-by-step explanation:
        
        *   `zcat` allows for reading the gzipped file without extracting it.
        *   [`awk`](https://www.geeksforgeeks.org/awk-command-unixlinux-examples/) is a handy string manipulating program
        *   `NR>1` means skip the first line since that line is the header.
        *   `tot_length+=$3-$2` is a cumulative variable that increases with the length of the region for each row. The values in column 2 is subtracted from column 3 and the sum is added to the variable `tot_length`.
        *   `tot_bases+=$x` is the cumulative value of your desired column.
        *   `END` just means do the following once the end of the file is reached.
        *   `print ` print the following to the command line.


        

## MultiQC
As you might have notice it takes quite a while to identify and find the different QC-values to track. To combat the never-ending file searching and enable easier tracking of QC-values the program [MultiQC](https://multiqc.info/) is often used. MultiQC aggregate the different QC-values into a single report. MultiQC only aggregate results from other programs (over [100 tools](https://multiqc.info/modules/) are automatically included, but you can also build your own tables or plots) and never does any calculation, therefor what values you can find and the layout of a MultiQC report varies a lot. It all depends on which programs you have run and input into MultiQC. MultiQC also allows you to configure almost all parts of the report with a config so a MultiQC-report does not always look like a "MultiQC-report".  

<br>
The report is a self-contained interactive `.html`-file where you can adapt/configure/highlight etc the plots and tables directly in your web-browser. It also allows for an easy export of table and plots if needed. 
!!! question "Question 9"
    :question: Open the provided MultiQC-report (`lab2/MultiQC_DNA_report.html`) and "play around" with the different tools and parameters. Once you feel ready upload a screen-print of the MultiQC report with sampleP highlighted.

!!! question "Question 10"
    :question: Fill in the same table as in Question 8 but for SampleD with the values found in the MultiQC report.

<!-- !!! question "Question 11"
    :question: Try generating a new MultiQC and using the data in lab2/Question10-folder where the fastp module is listed first. Then upload a screenshot to studium. 
    ```
    module load bioinfo-tools
    module load multiqc
    multiqc --config ${project_folder}/lab2/qc/multiqc_config.yaml ${project_folder}/lab2/qc/
    ```
    ??? tip
        Check the `multiqc_config.yaml` if there is something in there that can be changed? -->
<!-- 
extra assignments/question 9
edit a ready multiqc report mha config, tex ordningen eller lägga till module baserat på vad för filer som finns preppade?
-->

## Variant Quality
The next step after you have validated the quality of the sequencerun is variants. How do we know that a variant we see is real and not and not an artefact?

### BaseQ and MapQ
There are several different ways and steps to validate a variant. Most variant callers have build-in filtering steps that ensures that the variants reported are filtered or at least flagged if the confidence is low. One of the most basic way is to look at the [base quality](lab1.md)<!--find specific passage about phredsscore --> of the variant's alternative bases. Of course does baseQ vary depending sequencing [machine and/or version of said machine](https://en.wikipedia.org/wiki/FASTQ_format#Encoding). A good rule-of-thumb is that you always want at least a phred-score of at least 20 to be able to trust a base-call.

The other quality-value that is usually included are mapping quality, how well a read has aligned to exactly that specific region of the reference genome. Different aligners use different scoring algorithm and there for it is hard to compare quality values across programs. For some more in-depth reading see [here](http://www.acgt.me/blog/2014/12/16/understanding-mapq-scores-in-sam-files-does-37-42) and [here](https://davetang.org/muse/2011/09/14/mapping-qualities/) for example.

!!! question "Question 11"
    :question: Open `.bam` file called `sampleD_subset.bam` in IGV, and navigate to `chr17:7577114`. How many `T` calls have a basequality below 30Q?
    ??? tip
        Sort the position based on base (right click -> sort by -> base). Under view -> preferences -> alignments make make sure that  `shade mismatched bases by quality` is checked.

<!-- 14              -->

### Genotype Quality
Most variant callers have some sort of genotype quality scoring available for each variantcall. The problem here is that it is not standardized and each caller can have completely different values making them hard to compare. For example for the GATK produced vcf-files there are several different quality values, there is the `QUAL` column as well as the `GQ` and `PL` fields in the FORMAT column. For more in-depth info on the different vcf columns and annotation for GATK see more info [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035531692-VCF-Variant-Call-Format). While other programs might also use the `GQ` field but their definition is completely different. 

![gq_values](includes/images/gq_values.png)

If we compare two variantcallers ([GATK's Haplotypcaller](https://gatk.broadinstitute.org/hc/en-us/articles/360037225632-HaplotypeCaller) and [Google's Deepvariant](https://github.com/google/deepvariant)); both specialized on germline calling, both use the flag `GQ` and the `QUAL` column as a quality measurements, we see a clear difference on both of the quality scores for the same variants. That said, the genotype quality score is not useless, but actually a good quality measurement of a variant call, you just always have to know how the file were produced and what scoring thresholds are considered "good" for that specific program. 

### Background and Artifacts
Another step for validating you variant is eliminating the risk that the variant actually is a sequencing artefact or just background from a "messy" region.  

To identify artifacts you have to be able to separate the calls that look like a real call (quality, depth etc often look very good) but is actually a result of some sort of introduced bias from a disease causing variant. What further complicates the process is that since in sequencing we use organic enzymes and proteins we often introduce artifacts in the same region as the cell might have trouble processing the DNA such as long repetitive sequences or homopolymers. Some artifacts are easier to identify, for example sequencing chemistry can affect our ability to sequence certain areas.

!!! question "Question 12"
    :question: What is the most classic artifact introduced by 2-color chemistry in the Illumina sequencing?

To identify background noise you need to separate an actual call to the normal noise or distribution of basecalls. No techniques has 100 % base recall, and even if that was the case we still primarily sequence short-read today, there is always a chance that your read actually originates from somewhere else. Most background is below 5 % allel frequency otherwise it moves towards being an artefact, so for most high frequency variants we are safe. The problem arises when we want to be able to detect low frequency variants. To be able to detect low frequency variants we have to sequence deeper and have greater prior knowledge of the methods. 

Both background and artifacts it is a lot easier identify if a normal pool is available to compare your data with. A normal pool consists of a number (the more the better) of "healthy" samples processed identical (or as close as possible) as your sample. Small and big biases are always introduced what ever we do, therefore it is important to spread out the normal samples in several batches to mitigate batch-bias. And different sequencing machines use different chemistry you need a normal-pool for each machine type you plan to include. 

<!-- Something about how to calc AF? Jonas just a messy region. -->

<br>