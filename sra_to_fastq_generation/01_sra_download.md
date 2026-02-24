#!/bin/bash
# =============================================================================
# SCRIPT: download_sra.sh
# PURPOSE: Download raw sequencing data from NCBI's SRA database for
#          GSE266524, a TEA-seq dataset containing three modalities:
#          RNA (gene expression), ADT (protein/antibody), and ATAC (chromatin).
#
# WHAT IS SRA?
#   The Sequence Read Archive (SRA) is NCBI's repository for raw sequencing
#   data. Each dataset has:
#     - A GEO Series ID (GSE): the whole study         e.g. GSE266524
#     - A GEO Sample ID (GSM): one biological sample   e.g. GSM8249182
#     - An SRR Run ID (SRR):   one sequencing run       e.g. SRR28887172
#   One GSM can have multiple SRR runs (e.g., multiple lanes or replicates).
#
# WHAT IS A "SAMPLE" HERE?
#   In TEA-seq (and multimodal single-cell assays), each "sample" refers to
#   a different measurement modality captured from the same cells, NOT a
#   different patient or donor. The three modalities here are:
#     - RNA  → gene expression reads (scRNA-seq)
#     - ADT  → antibody-derived tags (protein level, CITE-seq style)
#     - ATAC → chromatin accessibility reads (scATAC-seq)
#
# WHAT IS prefetch?
#   `prefetch` is a tool from NCBI's sra-tools package. It downloads .sra
#   files from NCBI to your local machine or HPC storage. It is preferred
#   over wget/curl for SRA downloads because it:
#     - Automatically verifies file integrity after download
#     - Supports resuming interrupted downloads
#     - Handles NCBI's access protocols correctly
# 

# =============================================================================
# REQUIREMENTS
# =============================================================================
# This script requires the sra-tools package, which provides the `prefetch`
# command. Below are your installation options:
#
# My preferred option is— conda (RECOMMENDED)
#   conda is a package manager that handles dependencies automatically and
#   keeps tools isolated in environments, preventing conflicts.
#
#   Install sra-tools into a new environment:
#     conda create -n sratools -c bioconda -c conda-forge sra-tools
#     conda activate sratools
#
#   Or install into an existing environment (e.g., teaseq):
#     conda install -n teaseq -c bioconda -c conda-forge sra-tools
#     conda activate teaseq
# Verify installation:
#   prefetch --version
# =============================================================================

# =============================================================================
# HOW TO RUN THIS SCRIPT
# =============================================================================
# 1. Navigate to your target SRA directory:
#      cd /storage1/fs1/cooper_m/Active/r.sivani/Teaseq/paper_analysis/raw_data/mlin/gse266524/sra
#
# 2. Activate your conda environment:
#      conda activate teaseq
#
#
## Each SRR will be downloaded into its own subdirectory named after the SRR ID.
# For example: ./SRR28887172/SRR28887172.sra
# =============================================================================

# =============================================================================
# SAMPLE 1: GSM8249182 — RNA (Gene Expression)
# Modality: scRNA-seq reads capturing messenger RNA from individual cells.
# These reads are used to measure gene expression levels per cell.
# SRR runs: SRR28887172, SRR28887173, SRR28887174, SRR28887175
# make sure to change the directory where you have these srr files for the respective SRA
# applies for all the below 
# =============================================================================

```bash
echo "Downloading RNA sample (GSM8249182)..."

for i in SRR28887172 SRR28887173 SRR28887174 SRR28887175
do
    echo "  Fetching ${i}..."
    prefetch --output-directory . $i
    # --output-directory . : saves the downloaded .sra file in the current
    # working directory, inside a subfolder named after the SRR accession.
    # Example output: ./SRR28887172/SRR28887172.sra
done

echo "RNA download complete."
```

# =============================================================================
# SAMPLE 2: GSM8249183 — ADT (Antibody-Derived Tags / Protein)
# Modality: CITE-seq style antibody reads capturing surface protein levels.
# These reads quantify protein expression per cell using barcoded antibodies.
# SRR runs: SRR28887164, SRR28887170
# =============================================================================
```bash
echo "Downloading ADT sample (GSM8249183)..."

for i in SRR28887164 SRR28887170
do
    echo "  Fetching ${i}..."
    prefetch --output-directory . $i
done

echo "ADT download complete."
```

# =============================================================================
# SAMPLE 3: GSM8249184 — ATAC (Chromatin Accessibility)
# Modality: scATAC-seq reads capturing open chromatin regions in individual
# cells. These reads reveal which genomic regions are accessible (active).
# SRR runs: SRR28887143, SRR28887149, SRR28887176, SRR28887177
# =============================================================================
```bash
echo "Downloading ATAC sample (GSM8249184)..."

for i in SRR28887143 SRR28887149 SRR28887176 SRR28887177
do
    echo "  Fetching ${i}..."
    prefetch --output-directory . $i
done

echo "ATAC download complete."
echo "All samples downloaded successfully."
```
