# RNA-seq analysis of _Culex pipiens_

- Author: Biswash Ghimire
- Date created: 11/4/2026
- Date last modified: 11/8/26
- This was run at the Pitzer cluster of the Ohio Supercomputer Center (<https://www.osc.edu>)
- OSC working dir: /fs/ess/PAS3493/people/ghimire75/GA5
- GitHub repository: https://github.com/Ghimire99/GA5

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

```bash
for sample in S63 S64 S65 S66 S67 S68 S69; do
    sbatch scripts/trimgalore.sh \
        "$fastq_dir/${sample}_R1.fastq.gz" \
        "$fastq_dir/${sample}_R2.fastq.gz" \
        "$outdir/trimgalore"
done
```

## Step 2: Creating a transcriptome index with Salmon

```bash
sbatch scripts/salmon_index.sh \
    "$ref_transcripts" \
    "$outdir/salmon_index"
```

## Step 3: Pseudo-alignment and quantification with Salmon

```bash
for sample in S63 S64 S65 S66 S67 S68 S69
do
    sbatch scripts/salmon_quant.sh \
        "$outdir/trimgalore/${sample}_R1.fastq.gz" \
        "$outdir/trimgalore/${sample}_R2.fastq.gz" \
        "$outdir/salmon_index" \
        "$outdir/salmon_quant"
done
```

## Step 4: Summarizing the results of all steps with MultiQC

```bash
sbatch scripts/multiqc.sh \
    "$outdir" \
    "$outdir/multiqc"
```

## Software versions used

Each script prints the exact container tool version to its Slurm log at the end
of the run. Versions used in this run (confirmed from the logs):

- TrimGalore: 2.3.0 (container `trim-galore:2.3.0--a56e49e832976df4`)
- Salmon: 2.8.0 (container `salmon:2.8.0--3c08cbdc1b311e75`)
- MultiQC: 1.35 (container `multiqc:1.35--562c547b518ebe20`)