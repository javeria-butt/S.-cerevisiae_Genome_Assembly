# VGP Assembly Workflow: Logic and Implementation

This document outlines the procedural rationale behind the **Vertebrate Genomes Project (VGP)** pipeline, detailing the functional role of each phase in generating high-fidelity assemblies.

## Rationale for the VGP Framework

The VGP architecture is considered the gold standard for reference-grade genomics. While originally designed for complex vertebrates, applying it to *S. cerevisiae* provides several technical advantages:
* **Haplotype Resolution:** It generates phased, chromosome-level assemblies rather than collapsed sequences.
* **Operational Transparency:** Every step is reproducible and fully integrated within the Galaxy ecosystem.
* **Data Synergy:** It leverages the strengths of three distinct sequencing and mapping technologies.
* **Validated Outputs:** Built-in metrics ensure rigorous quality control throughout the process.

---

## Phase 1: Data Pre-processing & Genomic Profiling

### Read Curation
PacBio HiFi sequencing involves circularized DNA templates. Occasionally, the adapters used in this process are sequenced along with the genomic DNA. If left uncleaned, these adapters introduce "chimeric" artifacts, leading the assembler to create non-existent genomic connections.

**Cutadapt** is employed with stringent parameters (e.g., 35 bp minimum overlap) to identify and excise these adapter sequences, ensuring the input reads are purely biological.

### K-mer Based "Genome Preview"
Before initiating the assembly, we use **Meryl** and **GenomeScope2** to conduct a k-mer analysis. This provides a mathematical blueprint of the genome, allowing us to estimate:
* **Total Genome Size:** Establishing a baseline for assembly length.
* **Heterozygosity Levels:** Determining the divergence between chromosome copies.
* **Repeat Density:** Identifying potential regions of assembly complexity.
* **Sequencing Metrics:** Confirming that coverage and error rates meet the threshold for a high-quality build.

---

## Phase 2: De Novo Assembly Construction

### The Hifiasm Mechanism
**Hifiasm** is the primary engine for HiFi-based assembly. It operates through a specialized three-step workflow:
1.  **Haplotype-Aware Correction:** Fixes random sequencing errors while carefully preserving true biological variation.
2.  **String Graph Generation:** Creates a visual map of overlaps where heterozygous regions appear as structural "bubbles."
3.  **Hi-C Integration:** Utilizes Hi-C contact data to "untangle" these bubbles, effectively assigning sequences to their specific parent haplotypes (Hap1 vs. Hap2).

### Format Conversion
The output is initially generated as **GFA (Graphical Fragment Assembly)** files, which preserve the connectivity logic of the graph. We use **Gfastats** to linearize this data into standard **FASTA** format for compatibility with downstream analysis tools.

---

## Phase 3: Comprehensive Quality Assessment

### Functional Evaluation (BUSCO)
**BUSCO** assesses biological completeness by searching for near-universal single-copy orthologs. For yeast, we expect high recovery (>95%) of the 2,137 benchmark genes. This confirms the assembly is neither missing critical genes nor artificially duplicating genomic segments.

### Sequence Accuracy (Merqury)
While BUSCO looks at genes, **Merqury** looks at raw k-mers. It compares the k-mer profile of the original reads against the final assembly. This validates:
* **K-mer Completeness:** Ensuring the assembly contains all the sequence information present in the raw data.
* **Phasing Integrity:** Checking that the two haplotypes are accurately separated without "leakage" between them.

---

## Phase 4: Scaffolding to Chromosomal Scale

To bridge the gap between fragmented **contigs** and full **chromosomes**, we apply long-range scaffolding technologies.

### Physical Mapping via Bionano
**Bionano optical maps** visualize long DNA molecules labeled at specific sequence motifs. Because these maps span massive distances (hundreds of Megabases), they serve as a physical scaffold to:
* Correctly order and orient contigs.
* Accurately calculate the size of gaps between sequences.

### Proximity Ligation Scaffolding (YaHS)
**YaHS** utilizes Hi-C data, based on the principle that DNA sequences physically closer in the nucleus are more likely to be cross-linked. 
* It calculates interaction scores to link contig ends.
* It is particularly effective at resolving gaps in repetitive regions where optical maps may struggle.
* This step provides the final organizational structure for the chromosome-level assembly.

---

## Summary of Validation Milestones

| Milestone | Assessment Point | Primary Objective |
| :--- | :--- | :--- |
| **Post-GenomeScope2** | Step 3 | Validate estimated genome size and heterozygosity |
| **Post-Gfastats** | Step 5 | Review initial contiguity (N50 and contig count) |
| **Post-BUSCO** | Step 6 | Verify >95% ortholog recovery |
| **Post-Merqury** | Step 7 | Ensure >99% k-mer consensus and phasing quality |
| **Pre-Scaffolding Map** | Step 11 | Confirm internal consistency of contigs |
| **Final Assembly Map** | Step 14 | Final validation of chromosome-scale organization |
