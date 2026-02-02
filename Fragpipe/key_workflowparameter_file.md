# FragPipe DIA Workflow Documentation

**Workflow Name:** DIA_SpecLib_Quant  
**FragPipe Version:** 24.0    
**Purpose:** DIA data analysis with spectral library building and quantification

---

## 1. Workflow Overview

### Strategy

This workflow performs direct identification from DIA data and builds a spectral library for accurate quantification:

1. **MSFragger-DIA** → Direct peptide identification from DIA data
2. **MSBooster** → Deep learning predictions (RT, IM, spectra)
3. **Percolator** → Machine learning-based PSM validation
4. **ProteinProphet** → Protein-level inference
5. **EasyPQP** → Spectral library generation
6. **DIA-NN** → Library-based quantification (primary method)
7. **IonQuant** → MS1-based label-free quantification (complementary)

### Input Requirements

- **File format:** mzML (required for DIA-NN quantification)
- **Data type:** DIA mass spectrometry data
- **Instrument:** Compatible with high-resolution MS instruments
- **Database:** FASTA protein database (path: `C:\FragPipe\FragPipe-24.0\databases`)

### Key Outputs

- **Spectral library:** `library.tsv` (for DIA-NN)
- **Quantification:** `report.tsv` (DIA-NN main output)
- **Protein results:** `protein.tsv`, `peptide.tsv`, `psm.tsv`
- **MSstats format:** `MSstats.csv` (for downstream statistical analysis)

---

## 2. Critical Parameters

### Database Search (MSFragger)

| Parameter | Value | Why This Matters |
|-----------|-------|------------------|
| **Enzyme** | Strict Trypsin | Cleaves after K/R; matches sample preparation |
| **Missed cleavages** | 1 | Allows some incomplete digestion |
| **Peptide length** | 7-50 aa | Optimal range for MS detection |
| **Precursor tolerance** | ±20 ppm | Mass accuracy for precursor matching |
| **Fragment tolerance** | 20 ppm | Mass accuracy for fragment ion matching |
| **Mass calibration** | Enabled (mode 2) | Optimizes tolerances automatically; improves accuracy |

**Fixed Modification:**
- **Carbamidomethyl (C):** +57.021 Da - Standard cysteine alkylation from sample prep

**Variable Modifications (max 3 per peptide):**
- **Oxidation (M):** +15.995 Da - Common artifact
- **Acetylation (protein N-term):** +42.011 Da - Common biological modification
- **Phosphorylation (S/T/Y):** +79.966 Da - Key signaling modification
- **Deamidation (Q/C):** -17.027 Da - Artifact and biological modification
- **Deamidation (E):** -18.011 Da - Artifact and biological modification

**Rationale for modifications:** These capture the most common biological modifications and sample preparation artifacts. Phosphorylation is included for signaling pathway analysis.

---

### Identification & Validation

| Module | Status | Key Settings | Purpose |
|--------|--------|--------------|---------|
| **MSBooster** | ✓ Enabled | Predict RT, IM, MS/MS spectra<br>Model: DIA-NN | Adds deep learning features to improve PSM discrimination; increases sensitivity 10-30% |
| **Percolator** | ✓ Enabled | Min probability: 0.7<br>Post-processing: TDC | Machine learning rescoring; provides accurate FDR estimates |
| **ProteinProphet** | ✓ Enabled | Min probability: 0.5 | Protein-level inference from peptide identifications |

---

### Spectral Library Building (EasyPQP)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Fragment ions** | b, y | Standard ion series for CID/HCD fragmentation |
| **RT calibration** | noiRT | Uses common RT (ciRT) instead of iRT peptides |
| **IM calibration** | Automatic | Automatically selects best reference run |
| **Max mass error** | 15 ppm | Quality filter for library entries |

---

### Quantification

#### DIA-NN (Primary Quantification)

| Parameter | Value | Why This Setting |
|-----------|-------|------------------|
| **Q-value cutoff** | 0.01 | 1% FDR at precursor level |
| **Match-Between-Runs** | **DISABLED** | Not needed for DIA; library provides completeness |
| **Quantification strategy** | 3 (primary), 2 (secondary) | Optimized for library-based DIA quantification |
| **Generate MSstats** | Enabled | For downstream statistical analysis |
| **Outputs** | Peptide + modified peptide levels | Modification-aware quantification |

**Why MBR is disabled:** DIA data typically detects >90% of peptides across all runs. Library-based quantification provides high completeness without needing to transfer identifications between runs, reducing false positives.

#### IonQuant (Complementary MS1 Quantification)

| Parameter | Value | Purpose |
|-----------|-------|---------|
| **Label-free quant** | Enabled | MS1-based quantification |
| **MaxLFQ** | Enabled | Robust normalization algorithm |
| **Match-Between-Runs** | **DISABLED** | Same rationale as DIA-NN |
| **Normalization** | Median | Corrects for loading differences |
| **MS1 tolerance** | 10 ppm (m/z), 0.4 min (RT) | XIC extraction parameters |

---

## 3. Quality Control & FDR Settings

| Level | FDR Threshold | Method |
|-------|---------------|--------|
| **PSM** | 1% | Percolator (target-decoy competition) |
| **Peptide** | 1% | Percolator + ProteinProphet |
| **Protein** | 1% | ProteinProphet |
| **Ion (DIA-NN)** | 1% | DIA-NN q-value filtering |

**Decoy generation:** Reverse sequence with "rev_" prefix

---

## 4. Non-Standard Choices & Justifications

### Why Both DIA-NN AND IonQuant?

- **DIA-NN:** MS2-based library quantification (fragment-level) - primary for DIA
- **IonQuant:** MS1-based precursor quantification - provides complementary validation
- Using both increases confidence and enables comparison

### Why MSBooster + Percolator Instead of Just PeptideProphet?

- MSBooster adds powerful predictive features (RT, IM, spectra similarity)
- Percolator uses machine learning to optimally combine all features
- This combination is current best practice for DIA analysis
- Significantly improves sensitivity over traditional scoring

### Why These Specific Variable Modifications?

- **Limited to 5 modifications** to keep search space manageable
- **Phosphorylation:** Critical for cell signaling studies
- **Oxidation & Deamidation:** Most common artifacts that must be accounted for
- **Acetylation:** Common biological modification at protein N-terminus
- These cover >95% of expected modifications in typical proteomics experiments

---

## 5. Reproducibility Information

### Workflow Execution

```
FragPipe 24.0
MSFragger version: Latest with DIA support
DIA-NN: Integrated version
Percolator: Integrated version
```



## 6. Expected Outputs & How to Use Them

| Output File | Content | Use For |
|-------------|---------|---------|
| `library.tsv` | Spectral library | DIA-NN quantification input |
| `report.tsv` | DIA-NN quantification | Main quantitative results |
| `protein.tsv` | Protein-level quant | Protein expression analysis |
| `peptide.tsv` | Peptide-level quant | Peptide-level analysis |
| `MSstats.csv` | Statistical analysis format | Differential expression (MSstats) |
| `psm.tsv` | PSM-level results | Quality control |

---

## 7. Common Issues & Solutions

| Issue | Likely Cause | Solution |
|-------|--------------|----------|
| Low protein IDs | Wrong enzyme or modifications | Verify sample prep matches parameters |
| High %FDR | Database too large | Use species-specific database |
| Memory error | Too many peaks processed | Reduce `use_topN_peaks` to 500 |
| Slow processing | Large database | Database already sliced (enabled) |

---

## 8. References & Documentation

### Key Publications:

- **MSFragger-DIA:** Yu et al., *Nature Communications* 14:4154 (2023)
- **IonQuant:** Yu et al., *Mol Cell Proteomics* 20:100077 (2021)
- **Philosopher:** da Veiga Leprevost et al., *Nature Methods* 17:869 (2020)

### Official Documentation:

- **FragPipe:** https://fragpipe.nesvilab.org/docs/
- **DIA Tutorial:** https://fragpipe.nesvilab.org/docs/tutorial_DIA.html
- **Parameter Guide:** https://fragpipe.nesvilab.org/docs/tutorial_fragpipe.html

---

## 9. Parameter Summary Table

### Complete Parameter Settings for Issue #3

| Category | Parameter | Value | Impact on Results |
|----------|-----------|-------|-------------------|
| **Search** | Enzyme | Strict trypsin | Determines which peptides are searched |
| | Missed cleavages | 1 | Allows semi-tryptic peptides |
| | Precursor tolerance | ±20 ppm | Affects precursor matching stringency |
| | Fragment tolerance | 20 ppm | Affects fragment matching stringency |
| | Mass calibration | On (mode 2) | Automatically optimizes tolerances |
| **Mods** | Fixed: C | +57.021 Da | Accounts for alkylation |
| | Variable: M, N-term, STY, Q/C/E | See section 2 | Captures biological & artifact mods |
| **Validation** | MSBooster | Enabled | Improves sensitivity 10-30% |
| | Percolator | Enabled | Machine learning FDR control |
| | ProteinProphet | Enabled | Protein-level inference |
| **Library** | Fragment ions | b, y | Standard for HCD/CID |
| | RT calibration | noiRT | No iRT peptides needed |
| **Quant** | DIA-NN | Enabled, 1% FDR | Primary quantification |
| | MBR (both tools) | Disabled | Not needed for DIA |
| | IonQuant | Enabled | Complementary MS1 quant |
| | Normalization | Median | Corrects loading differences |

---


This documentation provides:

 **Overall workflow strategy** - DIA library building + quantification pipeline  
 **Critical parameters** - Search settings, modifications, validation, quantification  
 **Context for choices** - Why MBR disabled, why these mods, why MSBooster  
 **Reproducibility** - Complete settings and expected outputs  

### Key Decision Justifications:

1. **MBR disabled:** DIA data has high completeness; library-based quant doesn't need it
2. **MSBooster + Percolator:** Current best practice; improves sensitivity significantly
3. **Both DIA-NN and IonQuant:** Complementary methods provide validation
4. **5 variable modifications:** Balance between coverage and search space size
5. **1% FDR at all levels:** Standard stringent threshold for high-confidence results

---

