# Bioinformatics Challenge Exercises

These exercises cover a wide range of bioinformatics topics, including sequencing, RNA-seq, metagenomics, phylogenetics, population genomics, single cell sequencing, protein structure, CRISPR genome editing, metabolomics, and machine learning for omics. Figures for each challenge are placed in the right column to aid visualization.

---

## 1 RNA Sequencing Challenges

RNA sequencing is the focus keyword.

### Challenge 1

Quality assessment of RNA sequencing reads

Tasks

* Import paired FASTQ files
* Quantify total reads
* Compute per base quality and sequence length metrics
* Identify adapters and overrepresented sequences
* Generate one quality plot in the right column

Outputs

* Read counts table
* Quality plot
* Short summary

### Challenge 2

Transcriptome index preparation

Tasks

* Download reference transcript set
* Build index with Salmon or Kallisto
* Produce transcript table with identifiers and lengths

Outputs

* Indexed transcript directory
* Transcript summary table

### Challenge 3

Transcript quantification

Tasks

* Quantify transcript abundance using index
* Produce TPM and raw count matrices
* Generate TPM distribution plot

Outputs

* TPM matrix
* Raw count matrix
* TPM distribution plot

### Challenge 4

Normalization and exploratory analysis

Tasks

* Normalize raw counts
* Apply variance stabilizing transformation
* Create PCA plot
* Summarize sample clustering

Outputs

* Normalized matrix
* PCA plot
* Clustering interpretation

### Challenge 5

Differential expression analysis

Tasks

* Import metadata
* Fit differential model for two conditions
* Compute log fold changes and adjusted p values
* Produce significant gene list
* Generate volcano plot

Outputs

* Significant gene table
* Volcano plot
* Summary of up/downregulated genes

### Challenge 6

Functional interpretation

Tasks

* Perform gene set enrichment
* Map significant genes to pathways
* Produce enrichment table and bar plot

Outputs

* Pathway table
* Enrichment plot
* Functional interpretation

**Figures in right column**

* Figure 1 read quality profile
* Figure 2 TPM distribution
* Figure 3 PCA plot
* Figure 4 volcano plot
* Figure 5 pathway enrichment

---

## 2 Metagenomics Challenges

Metagenomics is the focus keyword.

### Challenge 1

Raw metagenomic read assessment

Tasks

* Load FASTQ files
* Quantify total reads per sample
* Compute per base quality and GC content
* Produce a quality plot

Outputs

* Read count table
* Quality plot
* Summary

### Challenge 2

Host read removal

Tasks

* Align reads to host genome
* Remove mapped reads
* Summarize retained read counts

Outputs

* Filtered FASTQ files
* Read retention table
* Interpretation

### Challenge 3

Taxonomic profiling

Tasks

* Run classifier on cleaned reads
* Produce relative abundance data
* Generate taxa proportion bar plot

Outputs

* Taxonomic table
* Bar plot
* Dominant taxa summary

### Challenge 4

Metagenomic assembly

Tasks

* Assemble reads into contigs
* Evaluate assembly with length and N50
* Produce contig length distribution plot

Outputs

* Contig FASTA
* Assembly metrics table
* Contig length plot

### Challenge 5

Genome binning

Tasks

* Group contigs into bins
* Assess bin completeness and contamination
* Summarize bin metrics

Outputs

* Bin directories
* Bin summary table
* Interpretation

### Challenge 6

Functional annotation

Tasks

* Predict genes from bins
* Map genes to functional categories
* Produce functional abundance table and plot

Outputs

* Gene predictions
* Functional table
* Functional plot

**Figures in right column**

* Figure 1 read quality
* Figure 2 taxa proportions
* Figure 3 contig length
* Figure 4 bin quality
* Figure 5 functional composition

---

## 3 Phylogenetics Challenges

Phylogenetics is the focus keyword.

### Challenge 1

Sequence acquisition and preprocessing

Tasks

* Import gene sequences
* Clean sequence names
* Check for ambiguous bases
* Remove low-quality sequences
* Produce sequence length table

Outputs

* Curated FASTA
* Length table
* Summary

### Challenge 2

Multiple sequence alignment

Tasks

* Align sequences
* Inspect alignment and gap distribution
* Produce alignment quality plot

Outputs

* Aligned FASTA
* Alignment statistics table
* Plot

### Challenge 3

Model selection

Tasks

* Evaluate substitution models
* Compare models
* Select best model
* Produce model score table

Outputs

* Model scores
* Justification of model choice

### Challenge 4

Tree inference

Tasks

* Build phylogenetic trees
* Compare methods
* Root trees with outgroup
* Produce tree figure

Outputs

* Tree files
* Rooted tree figure
* Clade interpretation

### Challenge 5

Bootstrap support assessment

Tasks

* Run bootstrap
* Quantify node support
* Produce support distribution plot

Outputs

* Bootstrap trees
* Support plot
* Low-support node table

### Challenge 6

Molecular clock analysis

Tasks

* Test clock-like behavior
* Apply relaxed clock if needed
* Estimate divergence times
* Produce time-calibrated tree

Outputs

* Calibrated tree
* Divergence table
* Temporal interpretation

**Figures in right column**

* Figure 1 alignment quality
* Figure 2 rooted tree
* Figure 3 bootstrap support
* Figure 4 time-calibrated tree

---

## 4 Population Genomics Challenges

Population genomics is the focus keyword.

### Challenge 1

Variant dataset preparation

Tasks

* Load VCF
* Filter low-quality variants and individuals
* Produce variant counts table

Outputs

* Filtered VCF
* Summary table

### Challenge 2

Population structure analysis

Tasks

* Convert variants to genotype matrix
* Run PCA
* Produce PCA plot
* Summarize clusters

Outputs

* PCA table
* PCA plot
* Interpretation

### Challenge 3

Allele frequency estimation

Tasks

* Compute allele frequencies
* Produce frequency table
* Generate allele frequency spectrum plot

Outputs

* Frequency table
* Plot
* Summary

### Challenge 4

Linkage disequilibrium patterns

Tasks

* Compute pairwise LD
* Generate LD decay curve

Outputs

* LD matrix
* LD decay plot
* Interpretation

### Challenge 5

Population differentiation

Tasks

* Compute Fst
* Produce Fst table
* Generate bar plot

Outputs

* Fst table
* Bar plot
* Summary

### Challenge 6

Demographic inference

Tasks

* Fit demographic model using allele frequency spectra
* Compare models
* Produce parameter estimates and model fit plot

Outputs

* Parameter table
* Model fit plot
* Interpretation

**Figures in right column**

* Figure 1 PCA
* Figure 2 allele frequency spectrum
* Figure 3 LD decay
* Figure 4 Fst bar plot
* Figure 5 demographic fit

---

## 5 Single Cell Sequencing Challenges

Single cell sequencing is the focus keyword.

### Challenge 1

Data import and quality control

Tasks

* Load raw count matrix
* Remove low-quality cells
* Produce retained cell table

Outputs

* Filtered matrix
* Table
* Summary

### Challenge 2

Normalization and feature selection

Tasks

* Normalize counts
* Identify highly variable genes
* Produce feature selection plot

Outputs

* Normalized matrix
* Feature table
* Plot

### Challenge 3

Dimensionality reduction

Tasks

* Apply PCA
* Compute neighborhood graph
* Generate UMAP plot

Outputs

* PCA scores
* UMAP plot
* Interpretation

### Challenge 4

Cell clustering

Tasks

* Cluster cells
* Produce cluster table
* Generate cluster plot

Outputs

* Cluster table
* Plot
* Description

### Challenge 5

Marker gene detection

Tasks

* Identify cluster markers
* Produce marker table
* Generate heatmap

Outputs

* Marker table
* Heatmap
* Summary

### Challenge 6

Trajectory inference

Tasks

* Build pseudotime trajectory
* Identify correlated genes
* Produce trajectory plot

Outputs

* Pseudotime values
* Correlated gene table
* Trajectory plot
* Interpretation

**Figures in right column**

* Figure 1 feature selection
* Figure 2 UMAP
* Figure 3 cluster visualization
* Figure 4 marker heatmap
* Figure 5 trajectory plot

---

## 6 Protein Structure Challenges

Protein structure analysis is the focus keyword.

### Challenge 1

Protein structure retrieval

Tasks

* Download PDB structures
* Filter by resolution and completeness
* Produce PDB summary table

Outputs

* PDB files
* Table
* Summary

### Challenge 2

Structure visualization

Tasks

* Load structure in visualization tool
* Highlight secondary elements
* Produce figure showing helices, sheets, loops

Outputs

* Structure figure
* Interpretation

### Challenge 3

Secondary structure analysis

Tasks

* Compute helix, sheet, coil proportions
* Produce bar plot

Outputs

* Table
* Bar plot
* Interpretation

### Challenge 4

Surface and binding site analysis

Tasks

* Identify surface residues and pockets
* Map binding sites
* Produce surface plot

Outputs

* Pocket annotation
* Plot
* Interpretation

### Challenge 5

Mutation impact assessment

Tasks

* Select amino acid substitutions
* Compute predicted stability changes
* Produce ΔΔG table and plot

Outputs

* Table
* Plot
* Interpretation

### Challenge 6

Comparative structural analysis

Tasks

* Align homologs
* Compute RMSD
* Generate heatmap

Outputs

* RMSD table
* Heatmap
* Interpretation

**Figures in right column**

* Figure 1 secondary structure
* Figure 2 composition bar plot
* Figure 3 binding site
* Figure 4 mutation ΔΔG plot
* Figure 5 structural similarity heatmap

---

## 7 CRISPR Genome Editing Challenges

CRISPR genome editing is the focus keyword.

### Challenge 1

Target selection and gRNA design

Tasks

* Identify target loci
* Design candidate gRNAs
* Produce gRNA table with efficiency scores

Outputs

* Table
* Summary

### Challenge 2

Off-target prediction

Tasks

* Predict off-targets
* Filter by mismatch tolerance
* Produce genome map

Outputs

* Off-target table
* Genome plot
* Interpretation

### Challenge 3

Editing efficiency simulation

Tasks

* Simulate cutting efficiency
* Predict indel distributions
* Produce histogram

Outputs

* Efficiency table
* Histogram
* Interpretation

### Challenge 4

Functional impact analysis

Tasks

* Annotate edited loci
* Predict frameshift or amino acid changes
* Produce functional effect table

Outputs

* Table
* Interpretation

### Challenge 5

Validation strategy

Tasks

* Propose PCR/sequencing assays
* Simulate expected results
* Produce validation figure

Outputs

* Assay figure
* Note on reliability

### Challenge 6

Comparative editing strategy

Tasks

* Compare alternative gRNAs
* Evaluate efficiency and off-targets
* Produce ranked gRNA table

Outputs

* Ranked table
* Interpretation

**Figures in right column**

* Figure 1 off-target distribution
* Figure 2 editing efficiency histogram
* Figure 3 functional impact
* Figure 4 validation assay diagram

---

## 8 Metabolomics Challenges

Metabolomics is the focus keyword.

### Challenge 1

Raw data import and quality assessment

Tasks

* Load MS/NMR data
* Check missing values and outliers
* Produce QC plot

Outputs

* Cleaned matrix
* Table
* QC plot

### Challenge 2

Normalization and scaling

Tasks

* Normalize and scale data
* Produce boxplot/density plot

Outputs

* Normalized matrix
* Plot
* Summary

### Challenge 3

Multivariate analysis

Tasks

* PCA/PLS-DA
* Visualize sample separation
* Summarize variance

Outputs

* Scores table
* Plot
* Interpretation

### Challenge 4

Differential metabolite analysis

Tasks

* Compare conditions
* Compute fold changes and p values
* Generate volcano plot

Outputs

* Significant metabolite table
* Volcano plot
* Summary

### Challenge 5

Pathway enrichment analysis

Tasks

* Map metabolites to pathways
* Perform enrichment
* Generate bar plot

Outputs

* Table
* Plot
* Interpretation

### Challenge 6

Integrated multi-omics analysis

Tasks

* Combine metabolomics with transcriptomics/proteomics
* Compute correlations
* Generate heatmap

Outputs

* Correlation table
* Heatmap
* Interpretation

**Figures in right column**

* Figure 1 QC plot
* Figure 2 normalized distribution
* Figure 3 PCA/PLS-DA
* Figure 4 volcano plot
* Figure 5 pathway enrichment
* Figure 6 multi-omics heatmap

---

## 9 Machine Learning for Omics Challenges

Machine learning for omics is the focus keyword.

### Challenge 1

Data import and preprocessing

Tasks

* Load omics dataset
* Handle missing values and outliers
* Split into training/test sets

Outputs

* Cleaned dataset
* Summary table

### Challenge 2

Feature selection

Tasks

* Apply univariate/multivariate methods
* Rank features
* Generate bar plot of top features

Outputs

* Feature table
* Plot
* Interpretation

### Challenge 3

Model training

Tasks

* Train model (RF, SVM, logistic regression)
* Optimize hyperparameters
* Produce training performance table

Outputs

* Trained model
* Table
* Note

### Challenge 4

Model evaluation

Tasks

* Evaluate on test set
* Compute accuracy, precision, recall, AUC
* Generate ROC curve

Outputs

* Test performance table
* ROC figure
* Interpretation

### Challenge 5

Interpretation and visualization

Tasks

* Extract feature importance/SHAP values
* Generate heatmap/summary plot

Outputs

* Feature importance table
* Plot
* Interpretation

### Challenge 6

Integration with pathways

Tasks

* Map predictive features to pathways
* Perform enrichment analysis
* Produce table and bar plot

Outputs

* Pathway table
* Plot
* Interpretation

**Figures in right column**

* Figure 1 feature importance
* Figure 2 ROC curve
* Figure 3 SHAP/heatmap
* Figure 4 pathway enrichment

