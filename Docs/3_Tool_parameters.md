# Exact Tool Parameters for VGP Pipeline

This section provides the specific settings used for every tool in the pipeline, ensuring the analysis is fully reproducible.

## Phase 1 — Data Preparation

### Step 1: Cutadapt
* **Single-end or Paired-end reads:** Single-end
* **FASTQ/A file:** `HiFi_collection (collection)`
* **Read 1 Adapters (Anywhere):**
    * **Adapter 1:** `ATCTCTCTCAACAACAACAACGGAGGAGGAGGAAAAGAGAGAGAT`
    * **Adapter 2:** `ATCTCTCTCTTTTCCTCCTCCTCCGTTGTTGTTGTTGAGAGAGAT`
* **Adapter Options:**
    * Maximum error rate: `0.1`
    * Minimum overlap length: `35`
    * Look for adapters in reverse complement: `Yes`
* **Filter Options:**
    * Discard Trimmed Reads: `Yes`

### Step 2a: Meryl (Count)
* **Operation type:** Count operations
* **Count operations:** Count: count the occurrences of canonical k-mers
* **Input sequences:** `HiFi_collection (trimmed)` [collection]
* **k-mer size selector:** Set a k-mer size
* **k-mer size:** `31`

### Step 2b: Meryl (Union-sum)
* **Operation type:** Operations on sets of k-mers
* **Operations:** Union-sum: return k-mers that occur in any input, set count to sum
* **Input meryldb:** `meryldb` [collection from 2a]

### Step 2c: Meryl (Histogram)
* **Operation type:** Generate histogram dataset
* **Input meryldb:** `Merged meryldb`

### Step 3: GenomeScope2
* **Input histogram file:** `meryldb histogram`
* **Ploidy:** `2`
* **k-mer length:** `31`
* **Output options:** Summary of the analysis: `Yes`
* **Advanced options:** Create `testing.tsv` file: `Yes`

---

## Phase 2 — Genome Assembly

### Step 4: Hifiasm
* **Assembly mode:** Standard
* **Input reads:** `HiFi_collection (trimmed)` [collection]
* **Hi-C partition:** Specify
    * **Hi-C R1 reads:** `Hi-C_dataset_F`
    * **Hi-C R2 reads:** `Hi-C_dataset_R`
* **Other settings:** Default

### Step 5a: Gfastats (GFA → FASTA)
* **Input GFA:** `Hap1 contigs graph` + `Hap2 contigs graph`
* **Tool mode:** Genome assembly manipulation
* **Output format:** `FASTA`
* **Generate initial set of paths:** `Yes`

### Step 5b: Gfastats (Statistics)
* **Input file:** `Hap1 contigs graph` + `Hap2 contigs graph`
* **Tool mode:** Summary statistics generation
* **Expected genome size:** `11747160`
* **Thousands separator:** `No`

### Step 5c: Data Post-processing
* **Column Join:** Default settings for joining Hap1 and Hap2 stats.
* **Search in textfiles:**
    * **that:** `Don't Match`
    * **Regular Expression:** `scaffold`
    * **Match type:** Case insensitive

---

## Phase 3 — Quality Control

### Step 6: BUSCO
* **Sequences:** `Hap1 contigs FASTA` + `Hap2 contigs FASTA`
* **Lineage data:** Use cached lineage (Busco v5 Datasets 2023-05-02)
* **Mode:** Genome assemblies (DNA)
* **Method:** Metaeuk
* **Lineage:** `saccharomycetes_odb10`
* **Outputs:** Short summary text + summary image

### Step 7: Merqury
* **Evaluation mode:** Default mode
* **k-mer counts database:** `Merged meryldb`
* **Assemblies:** Two assemblies (Hap1 & Hap2 FASTA)

---

## Phase 4 — Scaffolding

### Step 8b: Bionano Hybrid Scaffold
* **NGS FASTA:** `Hap1 contigs FASTA`
* **BioNano CMAP:** `Bionano_dataset`
* **Configuration mode:** `VGP mode`
* **Conflict filters:** Cut contig at conflict (for both Genome and Sequences)

### Step 8c: Concatenate Datasets
* **Concatenate:** `NGScontigs scaffold NCBI trimmed`
* **Insert:** `NGScontigs not scaffolded trimmed`

### Steps 9 & 10: BWA-MEM (Hi-C Mapping)
* **Reference:** `Hap1 assembly bionano` (Build index from history)
* **Reads:** Single-end (`Hi-C_dataset_F` for Step 9, `Hi-C_dataset_R` for Step 10)
* **Analysis mode:** 1. Simple Illumina mode
* **BAM sorting:** Sort by read names (`QNAME`)

### Step 10b: Filter and Merge
* **Input:** `BAM forward` + `BAM reverse`

### Steps 11a & 11b: Pretext (Initial)
* **PretextMap:** Sort = `Don't sort`
* **Pretext Snapshot:** Format = `png`, Grid = `Yes`

### Step 12: YaHS
* **Input sequences:** `Hap1 assembly bionano`
* **Alignment file:** `BAM Hi-C reads`
* **Enzyme sequence:** `CTTAAG`

### Steps 13 & 14: Final Mapping & Map Generation
* **Repeat BWA-MEM & Merge:** Use `YaHS Scaffolds FASTA` as reference.
* **Pretext Snapshot:** Generate final PNG with grid enabled.
