# PTM Site Mapping — Chromatin TF Proteins

**By Swanitha Gopu and Dr. Mark Ziemann**

## Key Finding
Three novel modification sites identified on EZHIP:

| Site | Modification | Fraction | Database Status |
|------|-------------|----------|----------------|
| S141 | Phosphorylation | Chromatin + Soluble | Novel — not in any database |
| S259 | Phosphorylation | Chromatin + Soluble | Confirms PRIDE entry — first chromatin detection |
| R302 | Arginine dimethylation | Chromatin | Novel — not in any database |

## Folder Structure
- workflows/ — FragPipe workflow files
- scripts/   — R Markdown analysis scripts
- results/   — TSV output files from R analysis
- figures/   — PyMOL structural figures
- data/      — Combined experiment profile and AlphaFold3 prediction

## Pipeline
1. FragPipe v23.1 open search — workflows/opensearchptm.workflow
2. R analysis — scripts/psm_combined_site.Rmd
3. Structural analysis — PyMOL with AlphaFold EZH2 + PDB 5HYN
4. AlphaFold 3 EZHIP-EZH2 complex — data/AlphaFold3_EZHIP_EZH2/

## Structural Results
- EZH2 AlphaFold vs PDB 5HYN: RMSD = 1.201 Å over 3578 atoms
- AlphaFold 3 complex: ipTM = 0.51, ranking score = 0.80
- Modification sites distal from EZH2 active site

## Next Steps
- Phospho enrichment TiO2/IMAC
- S141A and R302K mutagenesis
- Co-IP EZH2 with phospho-mutant EZHIP
- PRMT5 inhibitor GSK591 for R302 dimethylation

## Data Locations
- Raw files: HPC /home/swanitha.gopu/projects/proteomics/ptm_chromatin_analysis/
- Processed: Ziemann server /home/swanitha/
