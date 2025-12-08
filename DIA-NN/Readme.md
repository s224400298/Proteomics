# Proteomics Analysis with DIA-NN on Linux
By Swanitha and Dr. Mark Ziemann

## Introduction
Mass spectrometry-based proteomics using Data-Independent Acquisition (DIA) provides comprehensive protein identification and quantification with high reproducibility. DIA-NN is a state-of-the-art computational tool for processing DIA proteomics data, offering deep learning-based spectral prediction, high sensitivity, and accurate protein quantification.

This repository provides researchers with:

* Complete command-line workflow for DIA-NN analysis on Linux servers
* Reproducible pipeline from raw mass spectrometry data to statistical analysis
* Template R scripts for exploratory data analysis and differential expression
* Documentation for working with publicly available proteomics datasets
* Best practices for handling missing data and normalization


## Content of this Repository

* `README.md`: This comprehensive guide
* `Script`: Complete DIA-NN analysis commands for Linux
* `DIA-NN_EDA_Differential_Expression.Rmd`: R Markdown template for differential expression analysis


## Linux Server Setup
# Prerequisites

- Linux server access (Ubuntu recommended)
- Basic familiarity with command line operations
- .NET Runtime 8.0+ for reading Thermo .raw files
- Sufficient storage for raw data and analysis outputs 

### Install .NET Runtime (No Sudo Required)
- DIA-NN requires .NET Runtime to read Thermo .raw files. Install in your home directory:
```bash ## Download and install .NET SDK 8.0
cd ~
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 8.0 --install-dir ~/.dotnet

# Add to PATH
echo 'export PATH=$PATH:~/.dotnet' >> ~/.bashrc
echo 'export DOTNET_ROOT=~/.dotnet' >> ~/.bashrc
source ~/.bashrc

# Verify installation
dotnet --version
Download DIA-NN
bash# Create analysis directory
mkdir -p ~/projects/proteomics/diann-analysis
cd ~/projects/proteomics/diann-analysis

# Download DIA-NN 2.1.0 for Linux (most compatible)
wget https://github.com/vdemichev/DiaNN/releases/download/2.0/DIA-NN-2.1.0-Academia-Linux.zip

# Extract
unzip DIA-NN-2.1.0-Academia-Linux.zip

# Make executable
chmod +x diann-2.1.0/diann-linux

# Test installation
./diann-2.1.0/diann-linux
```

---

## **Using Publicly Available Data**

This workflow was tested with **PXD058340**, a dataset from Thermo Astral Orbitrap using data-independent acquisition:

- **Dataset**: Chromatin vs Soluble protein fractions from U2OS cells
- **Technique**: Data-Independent Acquisition (DIA) mass spectrometry
- **Instrument**: Thermo Astral Orbitrap
- **Samples**: 8 samples (4 chromatin-bound, 4 soluble fractions)
- **Proteins Identified**: 10,592 protein groups at 1% FDR
- **Data Availability**: PRIDE Archive (https://www.ebi.ac.uk/pride/archive/)

---

## **Data Structure Expected**
```
projects/proteomics/diann-analysis/
├── diann-2.1.0/                         # DIA-NN binaries
│   └── diann-linux                      # Main executable
├── raw_data/
│   ├── Chrom_01.raw                     # Chromatin fraction replicate 1
│   ├── Chrom_02.raw                     # Chromatin fraction replicate 2
│   ├── Chrom_03.raw                     # Chromatin fraction replicate 3
│   ├── Chrom_04.raw                     # Chromatin fraction replicate 4
│   ├── Sol_01.raw                       # Soluble fraction replicate 1
│   ├── Sol_02.raw                       # Soluble fraction replicate 2
│   ├── Sol_03.raw                       # Soluble fraction replicate 3
│   └── Sol_04.raw                       # Soluble fraction replicate 4
├── UP000005640_9606.fasta               # Human proteome database
├── library-from-fasta.predicted.speclib # Generated spectral library
├── library_generation.log               # Library generation log
├── search.log                           # Search log
└── results/
    ├── final_report.parquet             # Main results
    ├── final_report.pg_matrix.tsv       # Protein quantification matrix
    ├── final_report.pr_matrix.tsv       # Precursor intensity matrix
    ├── final_report.gg_matrix.tsv       # Gene group matrix
    ├── final_report.stats.tsv           # Analysis statistics
    └── final_report.log.txt             # Complete log
```

## Database Preparation
- Download Human Proteome from UniProt
```bash
cd ~/projects/proteomics/diann-analysis

# Download human proteome (UP000005640)
wget https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/reference_proteomes/Eukaryota/UP000005640/UP000005640_9606.fasta.gz

# Unzip
gunzip UP000005640_9606.fasta.gz
```
## Database Information:

- Organism: Homo sapiens (Human)
- Entries: ~20,000 protein sequences
- File size: ~14 MB
- Source: UniProt Reference Proteome

- Note: DIA-NN generates decoys automatically during library generation - no manual decoy creation needed.

## DIA-NN Analysis Workflow
- Complete Two-Step Pipeline
- Step 1: Generate Spectral Library from FASTA
  ```bash
  cd ~/projects/proteomics/diann-analysis
  # Generate spectral library with deep learning predictions
  nohup bash -c 'export PATH=$PATH:~/.dotnet && export DOTNET_ROOT=~/.dotnet && \
  ./diann-2.1.0/diann-linux \
  --lib "" \
  --threads 16 \
  --verbose 3 \
  --out-lib library-from-fasta.predicted.speclib \
  --gen-spec-lib \
  --predictor \
  --fasta UP000005640_9606.fasta \
  --fasta-search \
  --met-excision \
  --cut K*,R* \
  --missed-cleavages 2 \
  --unimod4' > library_generation.log 2>&1 &
- Step 2: Search Raw Files with Library
  ```bash
  # Search all raw files with Match Between Runs enabled
  nohup bash -c 'export PATH=$PATH:~/.dotnet && export DOTNET_ROOT=~/.dotnet && \
  ./diann-2.1.0/diann-linux \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Chrom_01.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Chrom_02.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Chrom_03.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Chrom_04.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Sol_01.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Sol_02.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Sol_03.raw \
  --f 20240911_AST0_NEO4_IAH_collab_SAG_U2OS_Frac_Control_Sol_04.raw \
  --lib library-from-fasta.predicted.speclib \
  --threads 8 \
  --out final_report.parquet \
  --qvalue 0.01 \
  --matrices \
  --met-excision > final_analysis.log 2>&1 &
  ```

# Monitor progress
 ```bash
 tail -f search.log
```
## **Key DIA-NN Parameters**

### **Library Generation Parameters**

| Parameter | Description | Default/Recommended |
|-----------|-------------|---------------------|
| `--fasta` | Protein sequence database | Required |
| `--fasta-search` | Enable FASTA digest | Required for library generation |
| `--gen-spec-lib` | Generate spectral library | Required |
| `--predictor` | Use deep learning predictions | Highly recommended |
| `--met-excision` | N-terminal methionine excision | Common for eukaryotes |
| `--cut K*,R*` | Trypsin specificity | Standard |
| `--missed-cleavages 2` | Allow up to 2 missed cleavages | Standard |

### **Search Parameters**

| Parameter | Description | Default/Recommended |
|-----------|-------------|---------------------|
| `--lib` | Spectral library path | Required |
| `--qvalue 0.01` | 1% FDR threshold | Standard |
| `--matrices` | Generate intensity matrices | Recommended |
| `--reanalyse` | Enable Match Between Runs (MBR) | Improves ID numbers |
| `--rt-profiling` | RT-dependent analysis | Recommended |

---

## **Output Files and Structure**

### **DIA-NN Output Directory**
```
diann-analysis/
├── library-from-fasta.predicted.speclib  # Spectral library
├── report.parquet                        # Main results (all PSMs)
├── report.pg_matrix.tsv                  # Protein quantification matrix
├── report.pr_matrix.tsv                  # Precursor intensity matrix
├── report.gg_matrix.tsv                  # Gene group matrix
├── report.stats.tsv                      # Analysis statistics
├── report.log.txt                        # Complete analysis log
├── *.quant                               # Per-file quantification
└── library_generation.log                # Library generation log
```

### **Key Output Files**
**Primary Quantification Matrix (report.pg_matrix.tsv):** 
- Contains quantified protein intensities across all samples
- Rows: Protein groups (UniProt IDs + Gene names)
- Columns: Sample intensities
- This is the main file for R statistical analysis

**Main Report (report.parquet):**
- Complete PSM-level data
- Can be read with R arrow package
- Contains detailed identification information

**Statistics File (report.stats.tsv):**
- Number of proteins/peptides per sample
- Quality control metrics
- Search confidence levels

**R-Based Statistical Analysis**
- Setup R Environment
```
r
# Install required packages
install.packages(c("arrow", "tidyverse", "limma", "pcaMethods", "pheatmap"))
```
**Exploratory Data Analysis and Differential Expression**
Execute the EDA workflow:
```r
# Basic EDA
setwd("~/projects/proteomics/diann-analysis/")
rmarkdown::render("DIA-NN_EDA_Differential_Expression.Rmd")
```
**Key EDA Features:**

1. **Data Quality Assessment**
   - Missing data patterns (10.48% overall)
   - Sample-wise completeness (86-93%)
   - Protein-wise completeness distribution

2. **Data Distribution Analysis**
   - Raw intensity distributions
   - Log transformation effects
   - Sample-to-sample variability

3. **Correlation Analysis**
   - Sample correlation heatmap
   - Within-group vs between-group correlations
   - Technical reproducibility assessment

4. **Principal Component Analysis**
   - Sample clustering visualization
   - Variance explained per PC
   - Quality control and outlier detection

5. **Normalization Pipeline**
   - Log10 transformation
   - Quantile normalization
   - Before/after comparison

**Expected EDA Results from Example Dataset:**
Dataset Dimensions:
- Proteins: 10,585
- Samples: 8

Missing Data Summary:
- Total missing: 8,873 values (10.48%)
- Chrom samples: 12-17% missing
- Sol samples: 6-7% missing

After Filtering (<3 NAs):
- Proteins retained: 8,919 (84%)
- Ready for statistical analysis

Sample Correlations:
- Chromatin replicates: 0.93-0.97
- Soluble replicates: 0.96-0.99  
- Between groups: 0.20-0.30

PCA Results:
- PC1: 93.91% variance (biological difference)
- PC2: 3.87% variance (technical variation)
- Clear separation between chromatin and soluble

**Key Differential Expression Analysis Features**
**Statistical Features:**

1. **Linear Modeling** (limma package)
   - Empirical Bayes moderation
   - High variance estimation
   - Accounts for technical variation

2. **Multiple Testing Correction**
   - Adjusted P-values for all proteins
   - Standard FDR threshold

3. **Effect Size Analysis**
   - Log2 fold-change quantification
   - Average expression levels
   - Statistical significance testing

4. **Visualizations**
   - Volcano plots (FC vs significance)
   - MA plots (expression vs FC)
   - Heatmaps of top proteins
   - QC validation plots

**Expected DE Results from Example Dataset:**
**Differential Expression Summary:**
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Total proteins analyzed:           8,919
- Significant (FDR < 0.05):          6,122 (68.6%)

- Chromatin-enriched:                2,521 (28.3%)
- High enrichment (FC > 2):        598
- Moderate enrichment (FC 1-2):  1,923

- Soluble-enriched:                  3,601 (40.4%)
- High enrichment (FC > 2):         40
- Moderate enrichment (FC 1-2):  3,561
- Not significant:                   2,797 (31.4%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Top Chromatin-Enriched Proteins:
1. Q9BZE4  (logFC: 1.60, adj.P: < 0.0001)
2. Q14690  (logFC: 1.62, adj.P: < 0.0001)
3. Q03701  (logFC: 1.85, adj.P: < 0.0001)
...

Histone Protein Validation:
- H2AZ1/H2AZ2 (logFC: 1.73, adj.P: < 0.0001) ✓
- MACROH2A2 (logFC: 1.80, adj.P: < 0.0001) ✓
- H4C1      (logFC: 1.83, adj.P: < 0.0001) ✓
- H2AX      (logFC: 1.54, adj.P: < 0.0001) ✓
All histones enriched in chromatin as expected!

## **Visualizations Generated**
**Exploratory Data Analysis Plots**
*Raw Data Distribution*
- Boxplots showing intensity distributions
- Log-transformed intensity distributions
- Sample-to-sample comparison

**Correlation Heatmap**
- Sample correlation matrix
- Color-coded by correlation strength
- Hierarchical clustering of samples

**Normalization Pipeline**
- Before normalization (raw)
- After log transformation
- After quantile normalization

 **PCA Plots**
- PC1 vs PC2 scatter plot
- Sample labels and groupings

 **Quality Control**
- Top 20 most abundant proteins
- Coefficient of variation distribution
- Density plots per sample

 **Differential Expression Plots**
 **Volcano Plot**
- Log2 fold-change vs -log10(P-value)
- Significance thresholds marked
- Color-coded by enrichment

 **Enhanced Volcano Plot**
- Fold-change cutoffs (|FC| > 1)
- Four categories: strong enrichment, significant, not significant
- Biological interpretation guide

 **MA Plot**
- Average expression vs fold-change
- Identifies intensity-dependent bias
- Highlights significant proteins

## **Common Issues and Troubleshooting**
**1. "Illegal instruction (core dumped)"**
*Problem: DIA-NN 2.2.0+ binary incompatible with older CPUs*
- Solution: Used DIA-NN 2.1.0 (more compatible)
```bash
# Download older version
wget https://github.com/vdemichev/DiaNN/releases/download/2.0/DIA-NN-2.1.0-Academia-Linux.zip
unzip DIA-NN-2.1.0-Academia-Linux.zip
chmod +x diann-2.1.0/diann-linux
./diann-2.1.0/diann-linux  # Test
```

**2. "dotnet: not found"**
*Problem: .NET Runtime not in PATH*
- Solution: Export PATH variables
```bash
# Temporary fix (current session)
export PATH=$PATH:~/.dotnet
export DOTNET_ROOT=~/.dotnet

# Permanent fix
echo 'export PATH=$PATH:~/.dotnet' >> ~/.bashrc
echo 'export DOTNET_ROOT=~/.dotnet' >> ~/.bashrc
source ~/.bashrc
```
## **Citation and References**
##### *Primary Citation:*
- Demichev V, Messner CB, Vernardis SI, Lilley KS, Ralser M. DIA-NN: neural networks and interference correction enable deep proteome coverage in high throughput. Nat Methods. 2020;17(1):41-44. doi:10.1038/s41592-019-0638-x

##### *Statistical Analysis*
- limma Package:Ritchie ME, Phipson B, Wu D, Hu Y, Law CW, Shi W, Smyth GK. limma powers differential expression analyses for RNA-sequencing and microarray studies. Nucleic Acids Res. 2015;43(7):e47. doi:10.1093/nar/gkv007
- Documentation: https://bioconductor.org/packages/devel/bioc/manuals/limma/man/limma.pdf

#### **Additional Resources**
- DIA-NN Official: https://github.com/vdemichev/DiaNN
- PRIDE Database: https://www.ebi.ac.uk/pride/
- UniProt: https://www.uniprot.org/










