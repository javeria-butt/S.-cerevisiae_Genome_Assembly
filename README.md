# High-Fidelity Genome Assembly of Saccharomyces cerevisiae S288C 

This project demonstrates a chromosome-level *de novo* assembly of the *S. cerevisiae* (strain S288C) genome. The workflow utilizes the **Vertebrate Genomes Project (VGP)** standards, executed via the Galaxy Europe platform. 

**Tutorial Source:** [Galaxy Training Network — VGP Genome Assembly](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html)  
**Platform:** Galaxy  
**Model Organism:** *Saccharomyces cerevisiae* S288C (synthetic diploid reads)

---

## 📋 Table of Contents
1. [Overview](#overview)
2. [Pipeline Summary](#pipeline-summary)
3. [Tools Used](#tools-used)
4. [Data](#data)
5. [Step-by-Step Workflow](#step-by-step-workflow)
6. [Key Results](#key-results)
7. [File Structure](#file-structure)
8. [Glossary](#glossary)
9. [References](#references)

---

## Overview
This repository documents the completion of the **Vertebrate Genomes Project (VGP)** genome assembly tutorial. The goal is to assemble a high-quality, chromosome-level genome using a multi-technology approach.

| Technology | Purpose |
| :--- | :--- |
| **PacBio HiFi** | Primary long-read assembly (10–25 kbp reads, ≥99% accuracy) |
| **Bionano Optical Maps** | Long-range scaffolding, resolves large structural regions |
| **Illumina Hi-C** | Chromosome-scale scaffolding using chromatin conformation |

The organism is *S. cerevisiae* S288C (~12 Mb haploid genome), ideal for benchmarking assembly quality.

---

## Pipeline Summary
```text
Raw HiFi reads
      │
      ▼
[Cutadapt] — adapter removal
      │
      ▼
[Meryl + GenomeScope2] — k-mer profiling → genome size, heterozygosity
      │
      ▼
[hifiasm (Hi-C mode)] — contig assembly → Hap1 & Hap2 GFA graphs
      │
      ▼
[gfastats + BUSCO + Merqury] — assembly QC
      │
      ▼
[Bionano Hybrid Scaffold] — optical map scaffolding
      │
      ▼
[BWA-MEM2 + YaHS] — Hi-C scaffolding → chromosome-scale assembly
      │
      ▼
[PretextMap + Pretext Snapshot] — final Hi-C contact map QC
```
## Tools Used

| Tool | Version | Purpose |
| :--- | :--- | :--- |
| **Cutadapt** | 4.4 | Adapter trimming / removal |
| **Meryl** | 1.3 | k-mer counting |
| **GenomeScope2** | 2.0 | Genome size and heterozygosity estimation |
| **hifiasm** | 0.19.8 | De novo phased contig assembly |
| **gfastats** | 1.3.6 | Assembly statistics |
| **BUSCO** | 5.5.0 | Gene completeness assessment |
| **Merqury** | 1.3 | k-mer based QV and completeness |
| **Bionano Hybrid Scaffold** | 3.7.0 | Optical map scaffolding |
| **BWA-MEM2** | 2.2.1 | Hi-C read mapping |
| **YaHS** | 1.2a.2 | Hi-C scaffolding |
| **PretextMap** | 0.1.9 | Contact map generation |

---

## Data

All input data is sourced from Zenodo:
* **HiFi Reads:** [50x Synthetic Reads](https://zenodo.org/record/6098306)
* **Hi-C Reads:** [Illumina PE Reads](https://zenodo.org/record/5550653)
* **Bionano Map:** [CMAP file](https://zenodo.org/records/5887339)

---

## Step-by-Step Workflow

### 1. Data Upload & Preprocessing
* **Action:** Grouped HiFi files into a "Dataset Collection".
* **Cutadapt:** Used to discard reads containing adapters (`ATCTCTCT...` sequence).
* **Logic:** Reads with internal adapters are likely chimeric and must be removed entirely to prevent downstream misassemblies.

### 2. Genome Profile Analysis
* **Meryl:** Counted k-mers using a k-mer size of 31.
* **GenomeScope2:** Estimated the haploid size at **~11.7 Mb** and heterozygosity at **~0.576%**. The model fit was **>93%**, confirming high-quality input data.

### 3. Assembly with hifiasm
* **Mode:** Hi-C phased mode (`--hic`).
* **Result:** Generated two haplotype-resolved graphs (**Hap1** and **Hap2**). This approach prevents the "collapsed" assembly issues common in diploid organism workflows.

### 4. Assembly Evaluation
* **gfastats:** Confirmed ~16-17 contigs per haplotype, aligning with the expected yeast karyotype.
* **BUSCO:** Demonstrated high gene completeness against the *Saccharomycetes* lineage database.
* **Merqury:** Verified high base-level accuracy (QV) and correct k-mer distribution between phased haplotypes.

### 5. Scaffolding with Bionano
* **Action:** Integrated in-silico sequence maps with physical Bionano optical maps.
* **Impact:** Resolved complex regions where sequence-based assembly stalled, specifically across large tandem repeats.

### 6. Scaffolding with Hi-C (YaHS)
* **Mapping:** Hi-C reads were mapped individually via **BWA-MEM2** to handle the variable insert sizes characteristic of chromatin capture.
* **YaHS:** Ordered and oriented the Bionano scaffolds into 17 final chromosome-scale units.
* **Correction:** Hi-C data successfully identified and flipped contigs that were misoriented during previous steps.

### 7. Final Evaluation
* The final Hi-C contact map produced via **Pretext** showed clear, concentrated triangular blocks along the diagonal.
* **Result:** Final assembly consists of **17 scaffolds** (representing the 16 nuclear chromosomes + 1 mitochondrial DNA).

---

## Key Results

| Metric | Value |
| :--- | :--- |
| **Estimated Genome Size** | ~11.7 Mb |
| **Final # Scaffolds** | 17 |
| **BUSCO Completeness** | High |
| **Reference Match** | Near-identical contact map topology |

---

## File Structure

```text
vgp-genome-assembly/
├── README.md                    ← Main documentation
├── Methods/                     ← Detailed tool parameters
├── Results/                     ← BUSCO summaries, stats, and plots
├── Parameters/                  ← Configuration files
├── Discussion/                  ← Analysis and interpretation
└── glossary.md                  ← Terms reference
```
## Glossary

* **Contig:** A contiguous, gap-free sequence resulting from the initial assembly.
* **Scaffold:** A series of contigs joined by gaps (represented by N's) based on long-range information.
* **Phasing:** The process of separating maternal and paternal genetic contributions in a diploid assembly.
* **k-mer:** A substring of length *k* used for statistical genomic profiling.
* **N50:** A statistical measure representing the contig length at which 50% of the total assembly length is contained.

---

## References

* **Cheng, H. et al. (2021).** Haplotype-resolved de novo assembly using phased assembly graphs with hifiasm. *Nature Methods*, 18, 170–175.
* **Rhie, A. et al. (2020).** Merqury: reference-free quality, completeness, and phasing assessment for genome assemblies. *Genome Biology*, 21, 245.
* **Zhou, C. et al. (2022).** YaHS: yet another Hi-C scaffolding tool. *Bioinformatics*, 39(1).
* **Galaxy Training Network.** [VGP Genome Assembly Tutorial](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html).
