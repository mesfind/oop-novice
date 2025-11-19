## Bioinformatics Challenges

High throughput sequencing is the focus keyword.
These challenges provide applied tasks in sequence handling, quality assessment, alignment, and variant interpretation.

### Challenge 1

Raw read quality assessment from malaria parasite FASTQ files

Tasks

* Load paired FASTQ files
* Count total reads per file
* Compute base position quality metrics
* Produce a quality plot
* Summarize GC distribution and adapter presence

Outputs

* Table of read counts for R1 and R2
* Quality plot in the right column
* Short scientific interpretation

### Challenge 2

Reference genome indexing

Tasks

* Obtain Plasmodium falciparum reference sequence
* Compress and index the file
* Build BWA index
* Produce a summary table of chromosomes and their lengths in megabases

Outputs

* Indexed reference directory
* Table with chromosome and size

### Challenge 3

Short read alignment pipeline

Tasks

* Align paired reads with BWA MEM
* Convert SAM to BAM
* Sort and index the BAM file
* Compute alignment statistics

Outputs

* Sorted and indexed BAM files
* Summary of mapped reads, unmapped reads, and read pairing rate

### Challenge 4

Variant detection and filtering

Tasks

* Generate mpileup
* Call variants with bcftools
* Filter variants using depth and quality thresholds
* Produce a clean VCF

Outputs

* Raw VCF
* Filtered VCF
* Table of variant counts per chromosome

### Challenge 5

Gene level annotation of variants

Tasks

* Import GFF annotation
* Match variant sites to gene regions
* Categorize coding and noncoding variants
* Summarize predicted variant effects

Outputs

* Variant to gene mapping table
* Scientific summary of coding versus noncoding distribution

### Challenge 6

Expanded sequencing example using regional data

Students receive synthetic Plasmodium sequencing reads from five regions in Ethiopia.
Each dataset contains known single nucleotide changes used to test comparison skills.

Tasks

* Combine data into tidy format
* Quantify mutation frequencies per region
* Compute pairwise genetic distance
* Produce a heatmap and summary table

Outputs

* Table of mutation frequencies
* Heatmap of regional distances
* Scientific interpretation of regional variation

### Challenge Figures

Place figures in the right column

* Figure 1 quality score profile
* Figure 2 alignment density along chromosome 14
* Figure 3 chromosome level variant distribution
* Figure 4 coding and noncoding proportion

