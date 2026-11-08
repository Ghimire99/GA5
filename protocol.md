# RNA-seq analysis of _Culex pipiens_

- Author: Biswash Ghimire
- Date created: 11/4/2026
- Date last modified: 
- This was run at the Pitzer cluster of the Ohio Supercomputer Center (<https://www.osc.edu>)
- OSC working dir: /fs/ess/PAS3493/people/ghimire75/GA5
- GitHub repository: <???>

## Prerequisites

- **Input files**:
  - Paired-end RNA-seq reads are in `../garrigos/fastq`

- **Software**:
  - The scripts run programs using Apptainer containers,
    with links contained inside the scripts.
  - If running at OSC, no software installations are needed.
  - If you're not at OSC, you need to have Apptainer and Slurm installed.

- Run all code from the working dir listed above.
  You don't need to create the output dirs, because the scripts will do so.

- Each script writes a Slurm log file (`slurm-<script>-<jobID>.out`) to the working dir.
  After the jobs of a step have finished, check these logs for errors,
  then move them into a `logs/` dir within that step's output dir.

## Set up: Download the reference transcriptome FASTA file from NCBI

```bash
# Downloaded on 11/4/26
wget -P data/ref "https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/016/801/865/GCF_016801865.2_TS_CPP_V2/GCF_016801865.2_TS_CPP_V2_rna.fna.gz"
```

## Set up: Set file paths

```bash
# Inputs:
fastq_dir=../garrigos/fastq
ref_transcripts=data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz

# Base output dir:
outdir=results
```

## Step 1: Read trimming with TrimGalore

for sample in S63 S64 S65 S66 S67 S68 S69; do
    sbatch scripts/trimgalore.sh \
        "$fastq_dir/${sample}_R1.fastq.gz" \
        "$fastq_dir/${sample}_R2.fastq.gz" \
        "$outdir/trimgalore"
done

## Step 2: Creating a transcriptome index with Salmon

sbatch scripts/salmon_index.sh \
    data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz \
    results/salmon_index

## Step 3: Pseudo-alignment and quantification with Salmon

for sample in S63 S64 S65 S66 S67 S68 S69
do
    sbatch scripts/salmon_quant.sh \
        results/trimgalore/${sample}_R1.fastq.gz \
        results/trimgalore/${sample}_R2.fastq.gz \
        results/salmon_index \
        results/salmon_quant
done

## Step 4: Summarizing the results of all steps with MultiQC

sbatch scripts/multiqc.sh \
    results \
    results/multiqc