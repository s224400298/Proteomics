# Proteomics Analysis with FragPipe and Docker

By Swanitha and Dr. Mark Ziemann

## Introduction

Mass spectrometry-based proteomics generates complex datasets requiring sophisticated computational pipelines for protein identification, quantification, and downstream statistical analysis. FragPipe is a comprehensive proteomics analysis pipeline that integrates multiple tools including MSFragger, IonQuant, and DIA-NN for data-independent acquisition (DIA) analysis.

This work provides researchers with:
* A functional Docker container with FragPipe and R-based analysis tools
* Step-by-step guides for customizing workflows for specific research projects
* Template R Markdown scripts for exploratory data analysis and differential expression
* Complete reproducible analysis from raw mass spectrometry data to biological insights

## Content of this Repository

* `README.md`: This comprehensive guide
* `Docker Guide`: Build instructions for the Docker image with FragPipe and R environment
* `Docker_file`:   To run the Fragpipe in a Docker container
* `decoys.R script`: Script to generate decoy databases 
* `proteomics_eda.Rmd`: R Markdown template for exploratory data analysis
* `differential_analysis.Rmd`: R Markdown template for differential expression analysis
* `Workflow_file.workflow`: Basic workflow file optimized for label-free quantification.
* `key_workflowparameter_file.md`: Reference guide for FragPipe workflow parameters covering Label-Free DDA, TMT/iTRAQ, SILAC, phosphoproteomics, DIA, and PTM analysis
  
  
## Docker Container Setup

### Prerequisites
- Docker installed on your system
- Basic familiarity with command line operations

### Available Docker Images
- **mziemann/fragpipe_mod1**: Initial version with MSFragger
- **mziemann/fragpipe_mod2**: ✅ **Recommended** - Updated version with Python compatibility fixes

### Pull and Start Docker Container

```bash
# Pull the latest Docker image
docker pull mziemann/fragpipe_mod2

# Start container with home directory mounted
docker run -it --rm -v $HOME:/home mziemann/fragpipe_mod2 bash
```

### Pre-installed Tools in Container
The Docker image includes:
- FragPipe 23.1 with complete dependency stack
- MSFragger-4.3 (`/fragpipe_bin/dependencies/MSFragger-4.3/MSFragger-4.3.jar`)
- IonQuant-1.11.11 (`/fragpipe_bin/dependencies/IonQuant-1.11.11/IonQuant-1.11.11.jar`)
- diaTracer-1.3.3 (`/fragpipe_bin/dependencies/diaTracer-1.3.3/diaTracer-1.3.3.jar`)

## Using Publicly Available Data

This workflow can be applied to publicly available proteomics datasets. For demonstration, we used PXD058340, which contains data from a Thermo Astral Orbitrap instrument using data-independent acquisition:
- **Dataset**: Chromatin vs Soluble protein fractions from U2OS cells
- **Technique**: Data-Independent Acquisition (DIA) mass spectrometry
- **Samples**: 8 samples (4 chromatin-bound, 4 soluble fractions)
- **Proteins**: 9,387 identified proteins for analysis

### Data Structure Expected
```
projects/proteomics/
├── raw_data/
│   ├── sample1.mzML
│   ├── sample2.mzML
│   └── ...
├── sampletest.workflow
├── sampletest.manifest.fp-manifest
├── UP000005640_9606_withDECOYS.fasta
└── workingdir01/ (output directory)
```

## Database Preparation

### Download and Prepare Human Proteome Database

```bash
# Generate decoy database using R script
Rscript decoys.R

# Download human proteome from UniProt
wget https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/reference_proteomes/Eukaryota/UP000005640/UP000005640_9606.fasta.gz

# Unzip database
gunzip UP000005640_9606.fasta.gz

# Set proper file permissions
chmod 644 UP000005640_9606.fasta UP000005640_9606_withDECOYS.fasta

# Replace DECOY prefix with FragPipe convention
sed -i 's/>DECOY_/>rev_/' UP000005640_9606_withDECOYS.fasta

# Verify replacement worked correctly
head -20 UP000005640_9606_withDECOYS.fasta | grep "rev_"
```

## FragPipe Configuration Files

### Workflow File (`sampletest.workflow`)
The workflow file defines the analysis parameters. 

### Setting Up Workflow Files in FragPipe GUI

1. **Start FragPipe GUI:**
   ```bash
   cd fragpipe-23.1/fragpipe-23.1/bin/
   ./fragpipe
   ```

2. **Configure Database:**
   - Navigate to "Config" tab
   - Set FASTA file path to your prepared database
     
3. **Setting uo Workflow:**
   - Navigate to "Workflow" tab
   - Load workflow and can use default workflow for conventional searches
   - Save to Custom folder
   
3. **Set Search Parameters:**
   - Go to "MSFragger" tab
   - Set precursor and fragment mass tolerances
   - Configure enzyme specificity (typically trypsin)
   
5. **Provide a FASTA Database:**
   - Browse and select protein Fasta File (e.g Enter UniProt proteome ID  UP000005640 for human)
   - Set normalization and filtering parameters
   - Verify decoy tag is set to "rev_"


### Manifest File (`sampletest.manifest.fp-manifest`)
The manifest file lists all input files and their experimental conditions:

```
/projects/proteomics/raw_data/Chrom_01.mzML	Chrom_01	
/projects/proteomics/raw_data/Chrom_02.mzML	Chrom_02	
/projects/proteomics/raw_data/Chrom_03.mzML	Chrom_03	
/projects/proteomics/raw_data/Chrom_04.mzML	Chrom_04	
/projects/proteomics/raw_data/Sol_01.mzML	Sol_01	
/projects/proteomics/raw_data/Sol_02.mzML	Sol_02	
/projects/proteomics/raw_data/Sol_03.mzML	Sol_03	
/projects/proteomics/raw_data/Sol_04.mzML	Sol_04	
```

### Setting Up Manifest Files in FragPipe GUI

1. **Add Files:**
   - In "Select Files" tab, click "Add files" or "Add folder"
   - Navigate to your raw data directory
   - Select all `.mzML` or `.raw` files

2. **Set Conditions:**
   - Assign experiment names to each file
   - Group replicates with consistent naming
   - Set biological conditions if performing comparative analysis

3. **Export Manifest:**
   - Click "Save as manifest" to export `.fp-manifest` file
   - Save in your project directory for command-line use

### Working in cloud server 
### Cloud Instance Setup

1. **Install Docker:**
   ```bash
   # Ubuntu
   sudo apt update
   sudo apt install docker.io

2. **Transfer Data:**
   ```bash
   # Using scp
   scp -r local_data/ user@cloud-server:/home/projects/proteomics/
   
   # Using rsync
   rsync -avz local_data/ user@cloud-server:/home/projects/proteomics/
   ```

3. **Run Analysis:**
   ```bash
   # Pull Docker image
   docker pull mziemann/fragpipe_mod2
   
   # Start analysis
   docker run -it -v /home/projects:/home/projects mziemann/fragpipe_mod2 bash
   ```


## FragPipe Analysis Execution

### Command Line Execution

```bash
# Navigate to FragPipe directory
cd fragpipe-23.1/fragpipe-23.1/bin/

# Execute headless analysis
./fragpipe --headless \
    --workflow /home/projects/proteomics/sampletest.workflow \
    --manifest /home/projects/proteomics/sampletest.manifest.fp-manifest \
    --workdir /home/projects/proteomics/workingdir02 \
    --config-tools-folder /fragpipe_bin/dependencies/
```

### Monitoring Progress

```bash
# Monitor log file
tail -f /home/projects/proteomics/workingdir02/fragpipe.log

# Check resource usage
top

# Monitor disk space
df -h /home/projects/proteomics/workingdir02/
```

## Output Files and Structure

### FragPipe Output Directory Structure
```
workingdir01/
├── fragpipe.log                    # Analysis log file
├── protein.tsv                     # Protein identifications
├── peptide.tsv                     # Peptide identifications
├── ion.tsv                         # Precursor ion data
├── dia-nn/
│   ├── report.pg_matrix.tsv       # ⭐ Main protein quantification matrix
│   ├── report.tsv                 # Detailed DIA-NN results
│   └── report.stats.tsv           # Analysis statistics
├── ionquant/
│   └── combined_proteins.tsv      # Alternative protein quantification
└── logs/                          # Detailed tool logs
```

### Key Output Files

**Primary Quantification Matrix (`report.pg_matrix.tsv`):**
- Contains quantified protein intensities across all samples
- Rows: Proteins (with UniProt IDs and gene names)
- Columns: Sample intensities
- **This is the main file for downstream R analysis**

**Protein Identifications (`protein.tsv`):**
- Protein identification confidence scores
- May show zero intensities (use `report.pg_matrix.tsv` instead)
- Contains sequence coverage and peptide counts

**Analysis Statistics (`report.stats.tsv`):**
- Number of proteins/peptides identified per sample
- Quality control metrics
- Search statistics and confidence levels

## R-Based Statistical Analysis

### Exploratory Data Analysis

Execute the EDA workflow:
```bash
# Inside Docker container
R -e 'rmarkdown::render("proteomics_eda.Rmd")'
```

**Key EDA Features:**
- **Data Quality Assessment**: Missing data patterns, sample completeness
- **Normalization Pipeline**: Log transformation + quantile normalization
- **Principal Component Analysis**: Sample clustering and batch effect detection
- **Quality Control**: Validation using histone proteins and known markers

**Expected Results from Example Dataset:**
- 9,387 proteins across 8 samples
- 7.55% missing data rate (acceptable for proteomics)
- 97.0% variance explained by PC1 (clear biological separation)
- >98% within-group correlations (excellent technical reproducibility)

### Differential Expression Analysis

Execute the DE analysis workflow:
```bash
# Inside Docker container
R -e 'rmarkdown::render("differential_analysis.Rmd")'
```

**Statistical Features:**
- **Linear Modeling**: limma-based analysis with empirical Bayes moderation
- **Multiple Testing**: FDR control using Benjamini-Hochberg correction
- **Effect Size Analysis**: Log fold-change quantification and significance testing
- **Visualizations**: Volcano plots, heatmaps, and quality control plots

**Expected Results:**
- 5,644 significantly differentially expressed proteins (67.4% of total)
- 2,277 proteins enriched in chromatin fractions
- 3,367 proteins enriched in soluble fractions
- Validation: All histone proteins enriched in chromatin (as expected)

### Pathway Enrichment Analysis

**Functional Analysis Pipeline:**
- **Database**: Reactome pathway annotations (downloaded automatically)
- **Method**: mitch package for over-representation analysis
- **Multiple testing**: FDR correction for pathway significance

**Expected Pathway Results:**
- 578 significantly enriched pathways (FDR < 0.05)
- **Chromatin-enriched**: RNA processing, DNA methylation, chromatin remodeling
- **Soluble-enriched**: Metabolic pathways, protein folding, biosynthesis

## Common Issues and Troubleshooting

### 1. FASTA Database Errors
**Error:** `FASTA file path is empty or the file is corrupted`
**Solution:**
- Verify database file exists and is readable
- Check decoy sequences are properly formatted
- Ensure file permissions are correct (`chmod 644`)

### 2. Python/SciPy Compatibility Issues
**Error:** `ImportError: cannot import name '_lazywhere' from 'scipy._lib._util'`
**Solution:**
- Use `mziemann/fragpipe_mod2` (latest version)
- Avoid `mziemann/fragpipe_mod1` which has outdated dependencies

### 3. Memory Issues in Cloud
**Error:** `Java heap space` or system hanging
**Solution:**
- Increase instance memory (minimum 16GB RAM)
- Reduce concurrent processes in FragPipe settings
- Process smaller batches of files

### 4. Zero Protein Intensities
**Issue:** `protein.tsv` shows zero intensities
**Solution:**
- Use `report.pg_matrix.tsv` from dia-nn output folder
- This file contains the actual quantified protein intensities
- `protein.tsv` may only contain identification confidence


