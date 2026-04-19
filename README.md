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

**Transformed Linear Profile:**
<img width="2000" height="2000" alt="genomescope_transformed_linear_plot" src="https://github.com/user-attachments/assets/e157a5ae-2830-4147-a08d-2f1dc9ecb0be" />
*Multiplying frequency × coverage amplifies the homozygous peak, making the diploid structure easier to see. Reports: len=11,743,432 bp, ab=0.58%, aa=99.4%, kcov=25, err=0.000943%.*

**Log Scale Views:**
Linear Log
<img width="2000" height="2000" alt="genomescope_log_plot" src="https://github.com/user-attachments/assets/de343865-bde1-407e-a7ee-42e94e5ffceb" />
Transformed Log
<img width="2000" height="2000" alt="genomescope_transformed_log_plot" src="https://github.com/user-attachments/assets/047b1f96-5951-42f0-a6fc-bdd9b516cf46" />

**Summary Statistics**
| Property | Min | Max |
| :--- | :--- | :--- |
| **Homozygous (aa)** | 99.4165% | 99.4241% |
| **Heterozygous (ab)** | 0.575891% | 0.583546% |
| **Genome Haploid Length** | 11,739,513 bp | 11,747,352 bp |
| **Genome Unique Length** | 11,016,399 bp | 11,023,756 bp |
| **Model Fit** | 92.5159% | 96.5191% |
| **Read Error Rate** | 0.000943190% | 0.000943190% |

## BUSCO — Gene Completeness
BUSCO searches for 2,137 conserved genes expected in every Saccharomycetes genome. High completeness (>95%) and low duplication (<5%) confirm a high-quality assembly with no false duplications.
**Hap1 — 95.9% Complete**
<img width="2000" height="1499" alt="busco_hap1" src="https://github.com/user-attachments/assets/72b7c913-27dd-4e72-ba7d-41047169d223" />
*C:95.9% [S:94.1%, D:1.8%], F:2.7%, M:1.4%, n:2137 — The bar is almost entirely light blue (complete single-copy genes). Very small dark blue (duplicated) and red (missing) segments confirm an excellent assembly.*

**Hap2 — 89.0% Complete**
<img width="2000" height="1499" alt="busco_hap2" src="https://github.com/user-attachments/assets/6a449c4d-5995-4b9d-89bc-9c65b9ac744a" />
*C:89.0% [S:87.6%, D:1.4%], F:2.5%, M:8.5%, n:2137 — Slightly lower completeness is expected for the alternate haplotype. Low duplication (1.4%) confirms no false duplications.*

**Comparison Table**
| Category | Hap1 | Hap1 % | Hap2 | Hap2 % |
| :--- | :--- | :--- | :--- | :--- |
| **Complete single-copy (S)** | 2010 | 94.1% | 1871 | 87.6% |
| **Complete duplicated (D)** | 40 | 1.8% | 31 | 1.4% |
| **Complete total (C)** | 2050 | 95.9% | 1902 | 89.0% |
| **Fragmented (F)** | 57 | 2.7% | 54 | 2.5% |
| **Missing (M)** | 30 | 1.4% | 181 | 8.5% |

## Merqury — K-mer Quality
Merqury compares k-mers in reads vs assembly — completely reference-free. Combined completeness of 99.9999% means virtually every k-mer in the reads is represented in one of the two assemblies.

**Completeness Statistics**
| Assembly | K-mers found | Total k-mers | Completeness |
| :--- | :--- | :--- | :--- |
| **Hap1** | 10,792,811 | 13,010,260 | 82.96% |
| **Hap2** | 11,611,483 | 13,010,260 | 89.25% |
| **Both** | 13,010,244 | 13,010,260 | 99.9999% |



