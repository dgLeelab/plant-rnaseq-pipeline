# plant-rnaseq-pipeline
A comprehensive RNA-seq analysis pipeline for plant transcriptomics, covering the full workflow from raw read QC to differential expression and GO enrichment analysis.

This pipeline has been applied to plant species including *Quercus* spp

## Pipeline overview
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
