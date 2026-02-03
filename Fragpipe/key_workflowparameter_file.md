# FragPipe Workflow Quick Reference

> **Version:** FragPipe 24.0  
> **Purpose:** Quick parameter changes for common proteomics experiments

---

## Choose Your Experiment

| Experiment | Changes Needed |
|------------|----------------|
| Label-Free DDA | ✅ Use default workflow |
| TMT/iTRAQ | 5 parameters |
| SILAC | 3 parameters |
| Phosphoproteomics | 2 parameters |
| DIA | 8 parameters |
| Oxidation Studies | Add mods |
| Missed Cleavages/Incomplete Digestion | 2 parameters |
| Glycoproteomics | 4 parameters |
| Ubiquitination | Add 1 mod |
| Acetylation (Histones) | Add 1 mod |
| Methylation | Add mods |
| Non-Tryptic Digestion | 2 parameters |

---

## 1. Label-Free DDA (Default)

**Base workflow:** `basicsearchworkflowfile` - No changes needed!

### Verify these settings:
```properties
ionquant.use-lfq=true
ionquant.mbr=1
msfragger.search_enzyme_name_1=stricttrypsin
msfragger.allowed_missed_cleavage_1=2
```

## 2. TMT/iTRAQ Labeling

### Change 5 parameters:

```properties
# 1-2. Switch to labeled mode
ionquant.use-lfq=false
ionquant.use-labeling=true

# 3-4. Enable TMT
tmtintegrator.run-tmtintegrator=true
tmtintegrator.channel_num=TMT-16

# 5. Add TMT to fixed mods (replace entire line)
msfragger.table.fix-mods=57.02146,C,true,-1; 229.162932,K,true,-1; 229.162932,n,true,-1
```

**TMT Options:** TMT-6, TMT-10, TMT-11, TMT-16, TMT-18, iTRAQ-4, iTRAQ-8

**Ratio compression fix:** `tmtintegrator.min_purity=0.7` (up from 0.5)

---

## 3. SILAC Labeling

### Change 3 parameters:

```properties
# 1-2. Switch to labeled mode
ionquant.use-lfq=false
ionquant.use-labeling=true

# 3. Define channels (example: Heavy R10/K8)
ionquant.light=
ionquant.heavy=K8;R10
```

**SILAC codes:** K4, K8 (lysine) | R6, R10 (arginine)

**For 3-state SILAC:** Add `ionquant.medium=K4;R6`

**Note:** Variable mods already in base workflow ✓

---

## 4. Phosphoproteomics

### Optional changes (phospho already included):

```properties
# For enriched samples
msfragger.max_variable_mods_per_peptide=5
msfragger.allowed_missed_cleavage_1=3
```

**Note:** Phosphorylation (79.96633,STY) is already in default workflow

**PTM localization:** `ionquant.locprob=0.75` (default, can increase to 0.9)

---

## 5. DIA

### Change 8 parameters:

```properties
# Enable DIA tools
diann.run-dia-nn=true
speclibgen.run-speclibgen=true

# MBR settings
ionquant.mbr=0
diann.mbr=true

# DIA-specific
diann.quantification-strategy=3
msfragger.use_topN_peaks=1000
msfragger.minimum_ratio=0.00
speclibgen.easypqp.rt-cal=noiRT
```

---

## 6. Oxidation Studies

### Add to variable mods:

```properties
# Already included: 15.9949,M (Met oxidation)

# Add these:
15.9949,W,false,2    # Trp oxidation
15.9949,P,false,2    # Pro oxidation
15.9949,H,false,2    # His oxidation
31.9898,M,false,1    # Met dioxidation

# Also increase:
msfragger.max_variable_mods_per_peptide=5
```

---

## 7. Missed Cleavages / Incomplete Digestion

**Use case:** Study proteolysis, poor digestion, or intentional incomplete digestion

```properties
# Increase missed cleavages
msfragger.allowed_missed_cleavage_1=5

# For semi-specific search (one end must be enzymatic)
msfragger.num_enzyme_termini=1
```

**Note:** Each missed cleavage ~doubles search time. Use 3-5 for studying missed lysines/arginines.

---

## 8. Glycoproteomics

### Enable glyco search:

```properties
# Enable PTM-Shepherd glyco mode
ptmshepherd.run-shepherd=true
ptmshepherd.run_glyco_mode=true
ptmshepherd.n_glyco=true
ptmshepherd.glyco_ppm_tol=50
```

**Note:** Requires glycan database. Use built-in or custom glycan composition file.

**Common N-glycans already searched:** HexNAc, Hex, NeuAc, Fuc

---

## 9. Ubiquitination

### Add to variable mods:

```properties
# Add GlyGly (K-ε-GG) - tryptic Ub remnant
114.0429,K,false,1
```

**Note:** This is the diGly signature after trypsin digestion of ubiquitinated proteins.

**Tip:** Use with `msfragger.allowed_missed_cleavage_1=3` as Ub sites may block cleavage.

---

## 10. Acetylation (Histones/Regulatory)

### Add to variable mods:

```properties
# Lysine acetylation (already has N-term acetylation)
42.0106,K,false,3
```

**Note:** N-terminal acetylation (42.0106,[^) already in default workflow.

**For histone studies:** Consider increasing to max 5 acetylations per peptide.

---

## 11. Methylation

### Add to variable mods:

```properties
# Lysine methylation
14.0157,K,false,2    # Monomethyl
28.0313,K,false,1    # Dimethyl
42.0470,K,false,1    # Trimethyl

# Arginine methylation
14.0157,R,false,2    # Monomethyl (symmetric/asymmetric)
28.0313,R,false,1    # Dimethyl
```

**Note:** Set `msfragger.max_variable_mods_per_peptide=5` for heavily methylated samples.

---

## 12. Non-Tryptic Digestion

### Change enzyme settings:

```properties
# For LysC
msfragger.search_enzyme_name_1=lysc
msfragger.search_enzyme_cut_1=K

# For GluC
msfragger.search_enzyme_name_1=gluc
msfragger.search_enzyme_cut_1=DE

# For Chymotrypsin
msfragger.search_enzyme_name_1=chymotrypsin
msfragger.search_enzyme_cut_1=FWYL
```

**Common enzymes:** `stricttrypsin`, `trypsin`, `lysc`, `argc`, `gluc`, `chymotrypsin`, `aspn`

**For dual enzyme:** Use `msfragger.search_enzyme_name_2` (advanced)

---

## Common Modifications Reference

### Fixed Mods (always present)
```properties
57.02146,C          # Carbamidomethyl (IAA alkylation)
71.037114,C         # Propionamide (alternative to IAA)
229.162932,K        # TMT on lysine
229.162932,n        # TMT on N-terminus
304.207146,K        # iTRAQ 8-plex on lysine
304.207146,n        # iTRAQ 8-plex on N-terminus
```

### Variable Mods (may be present)
```properties
15.9949,M           # Met oxidation
42.0106,[^          # N-term acetylation
79.96633,STY        # Phosphorylation
114.0429,K          # GlyGly (ubiquitin)
42.0106,K           # Lys acetylation
14.0157,K           # Lys monomethyl
28.0313,K           # Lys dimethyl
42.0470,K           # Lys trimethyl
0.984016,N          # Asn deamidation
0.984016,Q          # Gln deamidation
-17.0265,nQnC       # Pyro-glu (N-term Q/C)
-18.0106,nE         # Pyro-glu (N-term E)
203.0794,ST         # O-GlcNAc
```

---


## FDR Control

| Level | Parameter | Default | Stringent |
|-------|-----------|---------|-----------|
| Ion/PSM | `ionquant.ionfdr` | 0.01 (1%) | 0.001 (0.1%) |
| Peptide | `ionquant.peptidefdr` | 1.0 (off) | 0.01 (1%) |
| Protein | `ionquant.proteinfdr` | 1.0 (off) | 0.01 (1%) |
| PTM localization | `ionquant.locprob` | 0.75 (75%) | 0.9 (90%) |

**Recommendation:** Use 1% FDR for standard analyses, 0.1% for high-confidence hits.

---

## Performance Tuning

### For FASTER searches:
```properties
msfragger.allowed_missed_cleavage_1=1
msfragger.max_variable_mods_per_peptide=2
msfragger.isotope_error=0/1/2
```

### For MAXIMUM sensitivity:
```properties
msfragger.allowed_missed_cleavage_1=3
msfragger.max_variable_mods_per_peptide=5
msfragger.isotope_error=0/1/2/3
msbooster.run-msbooster=true
ionquant.mbr=1
```

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Too few IDs | Check enzyme, widen mass tolerance to ±20 ppm |
| Search too slow | Reduce to 3-4 variable mods, use 2 missed cleavages |
| No quantification | Verify `use-lfq` vs `use-labeling` setting |
| TMT ratio compression | Increase `tmtintegrator.min_purity` to 0.7 |
| Poor MBR | Check LC reproducibility, adjust `mbrrttol` |
| Memory error | Reduce `max_variable_mods_combinations` to 2000 |

---

## Database Selection

| Organism | UniProt ID | Source |
|----------|------------|--------|
| Human | UP000005640 | Swiss-Prot reviewed |
| Mouse | UP000000589 | Swiss-Prot reviewed |
| Rat | UP000002494 | Swiss-Prot reviewed |
| E. coli K-12 | UP000000625 | Swiss-Prot reviewed |
| Yeast | UP000002311 | Swiss-Prot reviewed |
| Contaminants | cRAP | [Download](https://www.thegpm.org/crap/) |

**Best practice:** Use reviewed (Swiss-Prot) + contaminants. Ensure decoys present (rev_ prefix).

---

## Key Parameter Summary

| Parameter | Purpose | Default | When to Change |
|-----------|---------|---------|----------------|
| `database.db-path` | FASTA location | User set | Always |
| `msfragger.search_enzyme_name_1` | Digestion enzyme | stricttrypsin | Match protocol |
| `msfragger.allowed_missed_cleavage_1` | Max missed sites | 2 | 3-5 for incomplete digestion |
| `msfragger.max_variable_mods_per_peptide` | Max var mods | 3 | 5 for heavily modified |
| `msfragger.precursor_mass_lower/upper` | MS1 tolerance | ±20 ppm | Match instrument |
| `msfragger.isotope_error` | C13 correction | 0/1/2/3 | Keep for high-res |
| `ionquant.mbr` | Match-between-runs | 1 | 0 for single runs |
| `ionquant.use-lfq` | Label-free quant | true | false for TMT/SILAC |
| `ionquant.use-labeling` | Labeled quant | false | true for TMT/SILAC |
| `ionquant.ionfdr` | FDR threshold | 0.01 (1%) | 0.001 for stringent |

---

## Pre-Flight Checklist

- [ ] Database path set: `database.db-path`
- [ ] Enzyme matches protocol: `search_enzyme_name_1`
- [ ] Fixed mods match sample prep (alkylation method)
- [ ] Variable mods appropriate for experiment
- [ ] Mass tolerances match instrument capabilities
- [ ] Quantification mode correct (LFQ vs labeled)
- [ ] TMT: Labels added to fixed mods
- [ ] SILAC: Channels defined correctly
- [ ] DIA: DIA-NN and library generation enabled
- [ ] FDR thresholds appropriate (1% standard)
- [ ] Test on 2-3 files before full dataset
- [ ] Sufficient disk space (10GB+ per sample)

---

## Workflow Templates

### Label-Free DDA
```bash
# Use: basicsearchworkflowfile (no changes)
```

### TMT-16plex
```bash
ionquant.use-lfq=false
ionquant.use-labeling=true
tmtintegrator.run-tmtintegrator=true
tmtintegrator.channel_num=TMT-16
# Add TMT to fix-mods
```

### SILAC 
```bash
ionquant.use-lfq=false
ionquant.use-labeling=true
ionquant.heavy=K8;R10
```

### DIA
```bash
diann.run-dia-nn=true
speclibgen.run-speclibgen=true
ionquant.mbr=0
diann.mbr=true
msfragger.use_topN_peaks=1000
```

### Phosphoproteomics
```bash
# Phospho already in base workflow
msfragger.max_variable_mods_per_peptide=5
msfragger.allowed_missed_cleavage_1=3
```

### Missed Cleavages Study
```bash
msfragger.allowed_missed_cleavage_1=5
msfragger.num_enzyme_termini=1  # semi-specific
```

---

## Resources

- **FragPipe Website:** [https://fragpipe.nesvilab.org/](https://fragpipe.nesvilab.org/)
- **GitHub:** [Nesvilab/FragPipe](https://github.com/Nesvilab/FragPipe)
---
