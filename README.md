# 🌿 Plant RNA-seq Analysis Pipeline

A comprehensive RNA-seq analysis pipeline for plant transcriptomics, covering the full workflow from raw read QC to differential expression and GO enrichment analysis.

This pipeline has been applied to plant species including *Quercus* spp. (oak trees).

---

## Pipeline Overview

```
Raw FASTQ reads
     │
     ▼
01. Prinseq        → Low-quality read & PCR duplicate removal
     │
     ▼
02. Trimmomatic    → Adapter trimming
     │
     ▼
03. HISAT2         → Splice-aware alignment to reference genome
     │
     ▼
04. StringTie      → Transcript assembly & abundance estimation (TPM/FPKM)
     │
     ▼
05. prepDE         → Count matrix generation
     │
     ▼
06. TPM Extraction → TPM expression matrix
     │
     ▼
07. TMM Normalization → Normalized expression matrix
     │
     ▼
08. PCA / Subcluster  → Expression pattern analysis
     │
     ▼
09. BLASTp → Arabidopsis homolog mapping
     │
     ▼
10. GO Analysis    → Functional enrichment (DAVID)
```

---

## Requirements

| Tool | Version | Purpose |
|------|---------|---------|
| [Prinseq-lite](https://sourceforge.net/projects/prinseq/) | 0.20.4 | Read QC & deduplication |
| [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) | 0.39 | Adapter trimming |
| [HISAT2](https://daehwankimlab.github.io/hisat2/) | 2.x | Spliced alignment |
| [SAMtools](http://www.htslib.org/) | 1.x | BAM processing |
| [StringTie](https://ccb.jhu.edu/software/stringtie/) | 2.x | Transcript quantification |
| [Trinity](https://github.com/trinityrnaseq/trinityrnaseq) | - | TMM normalization script |
| [BLAST+](https://blast.ncbi.nlm.nih.gov/doc/blast-help/) | - | Homolog search |
| [seqkit](https://bioinf.shenwei.me/seqkit/) | - | Sequence manipulation |
| Python | 3.x | TPM extraction script |

---

## Usage

### Step 1. Read QC — Prinseq

Remove low-quality reads and PCR duplicates.

```bash
# Run from directory containing raw FASTQ files
nohup bash module01_prinseq.sh &

# Quality thresholds applied:
# -min_len 50        : minimum read length
# -min_qual_score 5  : minimum per-base quality
# -min_qual_mean 15  : minimum mean quality
# -derep 14          : remove PCR duplicates
# -trim_qual_left/right 15 : quality trimming from both ends
```

> **QC check**: Proceed if "Good sequences (pairs)" ≥ ~30 million reads per sample.

A summary matrix (`cleaned.txt`) is automatically generated with columns:
`sample ID | total reads | total bases | cleaned reads`

---

### Step 2. Adapter Trimming — Trimmomatic

```bash
# Run from prinseq_good_results/
nohup bash module02_trimmomatic.sh &

# Parameters:
# ILLUMINACLIP:TruSeq3-PE.fa:2:30:10
# LEADING:10  TRAILING:10  MINLEN:50
```

---

### Step 3. Alignment — HISAT2

Build genome index first, then align.

```bash
# Build index (run once)
hisat2_extract_splice_sites.py genome.gtf > genome.ss
hisat2_extract_exons.py genome.gtf > genome.exon
hisat2-build -p 16 --exon genome.exon --ss genome.ss genome.fa genome_tran

# Align
nohup bash module03_hisat2.sh &

# Output: *_hisat2.sorted.bam per sample
```

> **QC check**: Alignment rate ≥ 80% is considered acceptable.

---

### Step 4. Transcript Quantification — StringTie

```bash
# Run from directory containing sorted BAM files
nohup bash module04_stringtie.sh &

# Output per sample: gene_abundances.tsv, transcripts.gtf
```

---

### Step 5. Count Matrix Generation

```bash
# Run from StringTie results directory
# Output: gene_counts_matrix.csv, transcript_count_matrix.csv
python3 module05_prepDE_after_Stringtie.py3
```

---

### Step 6. TPM Extraction

```bash
# Output: TPM.matrix
python3 module06_TPM_Extraction.py
```

---

### Step 7. TMM Normalization

```bash
# Output: TMM.matrix
bash module08_TMM_normalization.sh
```

---

### Step 8. PCA & Subcluster Analysis

```bash
bash module09_matrix_process.sh
```

---

### Step 9. Functional Annotation — BLASTp against Arabidopsis

Map differentially expressed genes to *Arabidopsis thaliana* homologs for GO analysis.

```bash
# Download Arabidopsis proteome and build BLAST DB
makeblastdb -in Arabidopsis_thaliana.TAIR10.pep.all.fa -dbtype prot -parse_seqids

# Extract protein sequences from subcluster gene lists
seqkit grep -f selected_genes.txt braker.geneuniq.aa > selected.pep.fa

# Run BLASTp
blastp \
  -query selected.pep.fa \
  -db Arabidopsis_thaliana.TAIR10.pep.all.fa \
  -max_target_seqs 1 \
  -outfmt "6 qseqid sseqid pident length evalue bitscore qcovs qlen slen" \
  -num_threads 32 \
  -out homolog_AT.tsv
```

---

### Step 10. GO Enrichment Analysis

Arabidopsis homolog IDs (`homolog_AT.tsv`) are submitted to [DAVID](https://david.ncifcrf.gov/) for functional enrichment analysis.

---

## Repository Structure

```
plant-rnaseq-pipeline/
├── README.md
├── scripts/
│   ├── module01_prinseq.sh
│   ├── module02_trimmomatic.sh
│   ├── module03_hisat2.sh
│   ├── module04_stringtie.sh
│   ├── module05_prepDE_after_Stringtie.py3
│   ├── module06_TPM_Extraction.py
│   ├── module08_TMM_normalization.sh
└── └── module09_matrix_process.sh
    
```
## Author

**Lee Daegeun**  
M.S. in Bioinformatics, Chungnam National University  
Skills: Python · R · Bash · NGS analysis · Plant genomics

---

## License

MIT License

