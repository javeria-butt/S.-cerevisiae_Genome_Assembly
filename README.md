# High-Fidelity Genome Assembly of Saccharomyces cerevisiae S288C 

This project demonstrates a chromosome-level *de novo* assembly of the *S. cerevisiae* (strain S288C) genome. The workflow utilizes the **Vertebrate Genomes Project (VGP)** standards, executed via the Galaxy Europe platform. 

By integrating long-read sequencing, chromatin conformation capture, and optical mapping, this pipeline achieves a highly contiguous and phased diploid assembly.

---

## 📑 Project Navigation
* [Core Methodology](#core-methodology)
* [Workflow Architecture](#workflow-architecture)
* [Data Sources](#data-sources)
* [Software Stack](#software-stack)
* [Key Metrics & QC](#key-metrics--qc)
* [Project Structure](#project-structure)

---

## Core Methodology
The assembly leverages the VGP "gold standard" approach, which utilizes three distinct data modalities to overcome the limitations of single-technology sequencing:

| Technology | Data Type | Primary Utility |
| :--- | :--- | :--- |
| **PacBio HiFi** | SMRT Reads (>Q20) | Initial contig generation and accuracy |
| **Illumina Hi-C** | Chromatin Capture | Haplotype phasing and long-range scaffolding |
| **Bionano** | Optical Maps | Physical-map validation and scaffolding |

---

## Workflow Architecture

### Stage 1: Pre-processing & Profiling
1. **Trimming:** Removal of sequencing adapters using `Cutadapt`.
2. **K-mer Analysis:** Generation of k-mer databases and histograms via `Meryl`.
3. **Estimation:** Statistical estimation of genome size and heterozygosity using `GenomeScope2`.

### Stage 2: Diploid Assembly
* **Contig Construction:** Executed `Hifiasm` in Hi-C phased mode to produce primary and alternate assemblies.
* **Conversion:** Processed GFA outputs into FASTA format with `Gfastats` for downstream analysis.

### Stage 3: Scaffolding & Refinement
* **Hybrid Scaffolding:** Integration of Bionano optical maps.
* **Hi-C Integration:** Mapping of Hi-C reads using `BWA-MEM`, followed by chimeric read filtering.
* **Proximity Scaffolding:** Final chromosome-level ordering using `YaHS` and visualization through `PretextMap`.

### Stage 4: Quality Assessment
* **Ortholog Analysis:** Evaluating gene completeness with `BUSCO`.
* **K-mer Validation:** Assessing assembly consensus and phasing via `Merqury`.

---

## Data Sources
All datasets were retrieved from **Zenodo** repositories:

* **HiFi Reads:** Synthetic 50× coverage (simulated via HIsim with a 2% mutation rate). [Zenodo: 6098306]
* **Hi-C Reads:** Paired-end Illumina data (SRR7126301). [Zenodo: 5550653]
* **Optical Map:** Bionano CMAP data. [Zenodo: 5887339]

---

## Software Stack
| Tool | Version | Role |
| :--- | :--- | :--- |
| `Cutadapt` | 5.2 | Adapter removal |
| `Meryl` | 1.3 | K-mer indexing |
| `GenomeScope2` | 2.0 | Genomic profiling |
| `Hifiasm` | 0.19.8 | *De novo* Assembler |
| `Gfastats` | 1.3.6 | Summary Statistics |
| `BUSCO` | 5.5.0 | Biological completeness |
| `Merqury` | 1.3 | K-mer QV & Completeness |
| `Bionano Hybrid` | 3.7.0 | Optical Scaffolding |
| `BWA-MEM` | 2.2.1 | Read Alignment |
| `YaHS` | 1.2a.2 | Hi-C Scaffolding |

---

## Key Metrics & QC
The final assembly successfully resolved the genome into its expected **16 chromosomes** with a total length of approximately **11.7 Mb**.

| Metric | Result |
| :--- | :--- |
| **Heterozygosity** | 0.58% |
| **Hap1 Contig N50** | ~922 kb |
| **Hap2 Contig N50** | ~923 kb |
| **BUSCO (Saccharomycetes)** | 95.9% (Hap1) |
| **Merqury Completeness** | 99.99% |
| **Assembly Level** | Chromosome-level |

---

## Repository Structure
```text
.
├── configs/          # Galaxy workflow configurations
├── reports/          # BUSCO, Merqury, and GenomeScope outputs
├── visualization/    # Hi-C contact maps (Pretext snapshots)
└── README.md         # Project documentation
```
## GenomeScope2 — Genome Profile

GenomeScope2 fits a mathematical model to the k-mer histogram to estimate genome properties before assembly begins.

**Linear Profile:**
<img width="2000" height="2000" alt="genomescope_linear_plot" src="https://github.com/user-attachments/assets/8d6868b8-b45b-4e6d-afa4-13ae1932906d" />
*Reading this plot: First peak at ~25× = heterozygous k-mers (one chromosome copy). Second peak at ~50× = homozygous k-mers (both copies). Orange = sequencing errors. Black model line fits blue observed data at 96.5% — results are trustworthy.*


