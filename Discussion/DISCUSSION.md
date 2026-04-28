# Genome Assembly Report: *Saccharomyces cerevisiae* S288C

## Overview
The primary goal of this tutorial was to assemble a high-quality, chromosome-level genome of **Saccharomyces cerevisiae S288C** from synthetic PacBio HiFi reads, using a multi-technology pipeline combining HiFi long reads, Bionano optical maps, and Illumina Hi-C chromatin conformation data. Overall, the assembly was highly successful and the final result closely matches the known reference genome, validating both the data quality and the VGP pipeline methodology.

---

## Genome Profiling
Before assembly, **GenomeScope2** estimated the haploid genome size at approximately **11.7 Mb** with a heterozygosity rate of **~0.576%** and a diploid sequencing coverage of **~50×**. 

* **Distribution:** The k-mer spectrum showed a clean bimodal distribution with peaks at ~25× (heterozygous) and ~50× (homozygous).
* **Model Fit:** Exceeded 93%, giving us strong confidence in these estimates.
* **Heterozygosity Note:** The low heterozygosity (~0.576%) was expected for a laboratory strain, meaning the assembler had relatively little haplotype divergence to work with during phasing.

---

## Contig Assembly
Using `hifiasm` in Hi-C phased mode produced two haplotype assemblies:
* **Hap1:** 16 contigs (~11.3 Mb)
* **Hap2:** 17 contigs (~12.2 Mb)

Given that the *S. cerevisiae* S288C reference genome is ~12.1 Mb (16 chromosomes + mtDNA), these values are very close to expectation. The fact that `hifiasm` produced only 16–17 contigs at this stage—before any scaffolding—reflects the power of HiFi reads (10–25 kbp), which allow the assembler to span most repetitive regions.

---

## Assembly Quality Metrics

### BUSCO
Results confirmed high gene completeness for both haplotypes. The very low proportion of duplicated BUSCO genes indicates that the Hi-C phasing worked effectively; collapsed haplotypes would have shown much higher duplication levels.

### Merqury
* **Spectra-asm:** Showed that homozygous k-mers (~50×) were correctly shared, while haplotype-specific k-mers (~25×) were split between Hap1 and Hap2.
* **Accuracy:** The grey region of low-frequency k-mers (sequencing errors) was minimal, consistent with the high accuracy of HiFi reads.

---

## Scaffolding

### Bionano Optical Maps
Bionano hybrid scaffolding improved assembly contiguity by merging contigs separated by structural regions that sequence-based assembly alone could not resolve. These maps provide physical distance information independent of sequence similarity.

### Hi-C Scaffolding with YaHS
The most dramatic improvement came from Hi-C scaffolding. 
* **Correction:** The pre-YaHS contact map showed some ordering artifacts; post-YaHS, the map showed clean, well-separated triangular blocks along the diagonal.
* **Utility:** YaHS identified contigs that Bionano had placed in the wrong orientation and corrected them.

---

## Final Assembly vs. Reference

| Metric | Our Assembly | Reference |
| :--- | :--- | :--- |
| **Total length** | ~12.1 Mb | ~12.1 Mb |
| **Number of scaffolds** | ~17 | 17 (16 chr + mtDNA) |
| **Contact map topology** | Clean diagonal blocks | Clean diagonal blocks |
| **BUSCO completeness** | High | High |

---

## Limitations and Considerations
* **Synthetic reads:** These lack the sequencing errors and library biases present in real-world data.
* **Genome Size:** *S. cerevisiae* is a small (~12 Mb), low-complexity genome; vertebrate genomes would present significantly higher challenges.
* **Purging:** Hi-C phased mode minimized false duplications, so `purge_dups` was not required.
* **Decontamination:** A production-grade assembly would still require a final step to remove microbial or organellar DNA.

---

## Conclusion
This tutorial demonstrated that the VGP assembly pipeline is capable of producing a near-reference-quality chromosome-level assembly. The combination of **HiFi long reads**, **Bionano optical maps**, and **Hi-C data** creates a powerful, complementary set of technologies that addresses the limitations of individual methods.
