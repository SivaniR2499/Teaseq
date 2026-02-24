# SRA to FASTQ Conversion Pipeline

This script converts downloaded `.sra` files from NCBI's Sequence Read Archive (SRA)
into gzip-compressed FASTQ format using `fastq-dump`, submitted as an LSF job array
on a high-performance computing (HPC) cluster.

---------------------------------------------------------------------------------------------------

## Background for Beginners

### What is an SRA file?
SRA (Sequence Read Archive) is the format NCBI uses to store raw sequencing data.
Before you can analyze sequencing data (e.g., alignment, quality control), you must
convert it to **FASTQ format** — a plain text file that stores each sequencing read
alongside its per-base quality scores.

### What is a GSM accession?
**GSM** (GEO Sample) IDs are unique identifiers assigned to individual samples in
NCBI's Gene Expression Omnibus (GEO) database. Each GSM may contain one or more
**SRR** (SRA Run) accessions, each representing an individual sequencing run. The
`.sra` files are organized by GSM → SRR in this pipeline.


## Directory Structure

Before running the script, your SRA files should be organized as follows:

gse266524/
├── gsm_list.txt # One GSM accession per line (e.g., GSM8249182)
├── sra/
│ ├── GSM8249182/
│ │ └── SRR.../
│ │ └── SRR....sra
│ ├── GSM8249183/
│ │ └── SRR.../
│ │ └── SRR....sra
│ └── GSM8249184/
│ └── SRR.../
│ └── SRR....sra
└── fastq2/
└── logs/ # Job output and error logs are written here

## Usage

> ⚠️ You **must** submit using `bsub < script.sh` (stdin redirection) so that LSF
> reads the `#BSUB` directives inside the file. Running `bsub bash script.sh` will
> ignore those directives and break the job array indexing.


The script reads the GSM accession for each job from `gsm_list.txt` using the
job array index. For example:
Line 1 → GSM8249182 (processed by job index 1)
Line 2 → GSM8249183 (processed by job index 2)
Line 3 → GSM8249184 (processed by job index 3)

## Requirements

| Tool | Purpose |
|------|---------|
| `sra-tools` (`fastq-dump`) | Converts `.sra` files to FASTQ format |
| `conda` | Environment management (environment: `teaseq`) |

```bash
bsub -G compute-cooper_m \
     -q general \
     -a 'docker(continuumio/anaconda3:2024.10-1)' \
     -R "rusage[mem=32GB]" \
     -M 16GB \
     < fastqdump_wrapper.sh
```

Argument	Meaning
--split-files |	Splits paired-end reads into _1.fastq.gz (Read 1) and _2.fastq.gz (Read 2). For single-end data, only _1.fastq.gz is produced.
--origfmt |	Preserves the original read names from the SRA file. Without this, fastq-dump reformats read names, which can cause issues with some downstream tools.
--gzip |	Compresses output files with gzip (.fastq.gz). This saves significant disk space since uncompressed FASTQ files are very large.
--outdir |	Directory where the output FASTQ files are written. Each GSM gets its own subdirectory.

Your output would look something like:
fastq2/<GSM_ID>/
├── <SRR_ID>_1.fastq.gz   # Read 1 (or single-end reads)
└── <SRR_ID>_2.fastq.gz   # Read 2 (paired-end only)
## note: the number of fastqs produced should match the reads per spot. For example when you click on teh metadata for a particular SRR --> you would see the number of reads per spot . If you see 4 reads per spot - then you can expect 4 fastq files for that SRR in the output folder

Notes
To add more samples, append additional GSM accessions to gsm_list.txt and
update the job array range in #BSUB -J accordingly (e.g., [1-5]%2).

The %2 concurrency limit can be adjusted based on cluster resource availability.

Ensure the all the paths for the directory exists before submission, or the job will
fail to write log files.

Just to be easy, it is always better to save the SRA, SRR and the fastqs like shown in the picture below-
Fastqs are saved in the fastq directory :
<img width="1131" height="600" alt="Screen Shot 2026-02-23 at 4 36 01 PM" src="https://github.com/user-attachments/assets/3fc03359-8400-4a28-92f6-3b2b7f7663cf" />
