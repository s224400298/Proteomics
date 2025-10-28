Proteomics Analysis with MaxQuant
By Swanitha and Dr. Mark Ziemann
Introduction
Mass spectrometry-based proteomics generates complex datasets requiring sophisticated computational pipelines for protein identification, quantification, and downstream statistical analysis. MaxQuant is a comprehensive quantitative proteomics software package that integrates the Andromeda search engine with advanced algorithms for accurate mass and time (AMT) tag-based protein identification and quantification.
This work provides researchers with:

A complete MaxQuant analysis pipeline from raw data to results
Step-by-step guides for label-free quantification (LFQ) and SILAC experiments
Template R scripts for exploratory data analysis and differential expression
Complete reproducible analysis from raw mass spectrometry data to biological insights

Content of this Repository

README.md: This comprehensive guide
mqpar_template.xml: MaxQuant parameter file template
maxquant_eda.Rmd: R Markdown template for exploratory data analysis
maxquant_differential_analysis.Rmd: R Markdown template for differential expression analysis


MaxQuant Installation
System Requirements

Operating System: Windows 10/11 or Linux (via Mono or .NET Core)
RAM: Minimum 16GB, recommended 32GB+
Disk Space: 50GB+ for analysis outputs
Software Dependencies:

.NET Framework 4.8 (Windows) or .NET 8.0 (Linux)
For Linux: Mono or .NET 8 runtime



Installation Steps
Windows:

Download MaxQuant from https://www.maxquant.org/
Extract the ZIP file to desired location (e.g., C:\MaxQuant)
Double-click StartMaxQuant.bat to launch GUI

Linux Server:
MaxQuant requires .NET 8 runtime. Choose one of these installation methods:
Option 1: Use MaxQuant's Installation Script (Recommended, No sudo)
bash# Download and extract MaxQuant
# MaxQuant includes a helper script to install .NET locally

cd MaxQuant_v2.7.3.0/

# Make script executable
chmod +x automated_installation.sh

# Run installation (downloads and installs .NET 8.0.302 locally)
./automated_installation.sh ./

# Reload shell configuration
source ~/.bashrc

# Verify installation
dotnet --version

Option 2: System-Wide Installation (Requires sudo)
# Install .NET 8 runtime
sudo apt update
sudo apt install dotnet-runtime-8.0

Option 3
# Or install locally without sudo
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --version 8.0.302 --install-dir ./dotnet8/

# Add to PATH
export PATH=$HOME/dotnet8:$PATH

# Verify installation
dotnet --version

---

## **Using Publicly Available Data**

This workflow can be applied to publicly available proteomics datasets. For demonstration, we used **PXD058340**, which contains data from a Thermo Astral Orbitrap instrument:

- **Dataset**: Chromatin vs Soluble protein fractions from U2OS cells
- **Technique**: Data-Dependent Acquisition (DDA) mass spectrometry
- **Samples**: 8 samples (4 chromatin-bound, 4 soluble fractions)
- **Proteins**: 6,693 identified proteins after filtering

---

## **Data Structure**
```
projects/maxquant/
├── raw_data/
│   ├── RAWFile1.raw
│   ├── RAWFile2.raw
│   └── ...
├── UP000005640_9606.fasta
├── mqpar.xml
└── combined/
    └── txt/  (output directory)

Database Preparation
Download Human Proteome Database
bash# Download from UniProt
wget https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/reference_proteomes/Eukaryota/UP000005640/UP000005640_9606.fasta.gz

# Unzip
gunzip UP000005640_9606.fasta.gz

# Set permissions
chmod 644 UP000005640_9606.fasta
Note: MaxQuant automatically generates decoy sequences internally, so no manual decoy generation is needed.

MaxQuant Configuration
GUI Setup (Windows/Local)

Start MaxQuant:

bash   cd MaxQuant_v2.7.3.0
   StartMaxQuant.bat  # Windows Run as administrator

Load Raw Files:

Click "Load" → Select .raw files
Files will appear in the experiment table


Set Experiment Names:

Assign meaningful names (e.g., Chrom_01, Sol_01)
Group replicates appropriately

Set Search Parameters:
In Group Specific Parameters: Specify labels
Type: Standard 
Multiplicity: 1 (For LFQ)
Enzyme: Trypsin/P
Missed cleavages: 2
Fixed modifications: Carbamidomethyl (C)
Variable modifications: Oxidation (M), Acetyl (Protein N-term)
Main search tolerance: 4.5 ppm
MS/MS tolerance: 20 ppm (FTMS)

Configure FASTA Database:

Navigate to "Global parameters" → "Sequences"
Add FASTA file: UP000005640_9606.fasta
Set identifier parse rule: >[^\|]*\|(.*?)\|

Quantification Settings:

Label-free quantification (LFQ): ☑ Enabled
Match between runs: ☐ Disabled (optional)
iBAQ: ☐ Disabled (optional)


Save Configuration:

"File" → "Save parameters" → mqpar.xml




Command Line Setup (Linux Server)
For Linux server analysis, you need to:

Prepare mqpar.xml on Windows first (using GUI)
Transfer files to server:

bash   # Transfer raw files
   scp -r raw_data/ user@server:/path/to/maxquant/

   # Transfer FASTA
   scp UP000005640_9606.fasta user@server:/path/to/maxquant/

   # Transfer parameter file
   scp mqpar.xml user@server:/path/to/maxquant/

Update file paths in mqpar.xml:

bash   # Use MaxQuant command-line tool to update paths
   dotnet MaxQuantCmd.dll mqpar.xml --changeFolder mqpar_linux.xml /path/to/fasta/ /path/to/raw/

Running MaxQuant
GUI Execution (Windows)

Click "Start" button in MaxQuant GUI
Monitor progress in the log window
Analysis typically takes 1-3 hours per file

Command Line Execution (Linux)
bash# Navigate to MaxQuant bin directory
cd MaxQuant_v2.7.3.0/bin/

# Run MaxQuant
nohup dotnet MaxQuantCmd.dll ~/mqpar_linux.xml > ~/maxquant.log 2>&1 &

# Monitor progress
tail -f ~/maxquant.log

# Check if still running
ps aux | grep MaxQuant
```

---

## **MaxQuant Output Files**

### **Output Directory Structure**
```
combined/
├── txt/
│   ├── proteinGroups.txt          # ⭐ Main protein quantification
│   ├── peptides.txt                # Peptide-level data
│   ├── evidence.txt                # All peptide-spectrum matches
│   ├── summary.txt                 # Run statistics
│   ├── parameters.txt              # Analysis parameters
│   ├── msms.txt                    # MS/MS spectra details
│   └── modificationSpecificPeptides.txt
├── proc/                           # Processing files
└── andromeda/                      # Search engine files
Key Output Files
FileDescriptionproteinGroups.txtPrimary file for downstream analysis. Contains LFQ intensities, identification info, and protein groupspeptides.txtPeptide-level quantification and modificationsevidence.txtAll identified peptide-spectrum matches with retention timessummary.txtQC metrics: proteins/peptides identified per sample

R-Based Statistical Analysis
Exploratory Data Analysis
Execute the EDA workflow:
r# In R or RStudio
rmarkdown::render("maxquant_eda.Rmd")
Key EDA Features:

Data Quality Assessment: Missing data patterns, sample completeness
Normalization Pipeline: Log transformation + quantile normalization
Principal Component Analysis: Sample clustering and biological separation
Quality Control: Histone protein validation

Expected Results from Example Dataset:

6,693 proteins after filtering
Clear separation between chromatin and soluble fractions in PCA
High within-group correlations 





Common Issues and Troubleshooting
1. Out of Memory Errors
Error: Java heap space or system hanging
Solution:

Increase RAM allocation in MaxQuant settings
Process fewer files simultaneously
Use server with more RAM (32GB+)

2. .NET/Mono Compatibility (Linux)
Error: Unable to load DLL or framework errors
Solution:

Use .NET 8 instead of Mono
Install correct .NET runtime version
Check library dependencies

3. Invalid Format / Line Ending Issues
Error: invalid format when reading key files
Solution:
bash# Fix Windows line endings
dos2unix mqpar.xml
# OR
sed -i 's/\r$//' mqpar.xml
4. Empty proteinGroups.txt
Issue: Analysis completes but no proteins identified
Solution:

Check raw files are not corrupted
Verify FASTA database is correct format
Check search tolerances aren't too strict
Review summary.txt for search statistics

5. Disk Space Issues (Linux)
Error: Analysis stops at MS/MS preparation
Solution:
bash# Check disk space
df -h

# Clean up temporary files
rm -rf combined/proc/*

Performance Tips

CPU threads: Set to number of CPU cores 
RAM: Allocate 4-8GB per thread
Disk: Use SSD for faster I/O
File size: Large files (>5GB) may take 2-3 hours each


