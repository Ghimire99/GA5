<!--
GA5 README template.
- Add your answers to each question's section below.
- For each question where we want you to add your commands,
  put the command(s) you ran in the terminal in the corresponding ```bash code block.
- Where we want to see a command's output, there is a plain ``` code block
  right below the command block: paste the output there.
- The code to run each step of the workflow goes in `protocol.md`, not here:
  you don't need to repeat it in this file.
- You don't need to paste the contents of your scripts into this file:
  we will look at the scripts in your `scripts` dir directly
  (so make sure you commit them!).
- Delete instructional comments like this one as you go.
-->

# GA5 answers - Biswash Ghimire

------------------------------------------------------------------------

## Part A: Your project dir, Git repo, and protocol

### 1: Fill in the top of the protocol

<!-- No answer needed here -- we check `protocol.md` directly. -->

### 2: Markdown formatting

<!-- No answer needed here -- we check the formatting of both Markdown files. -->

### 3: Create subdirs, and copy the scripts

```bash
mkdir -p scripts data/ref results
```

### 4: Initialize a Git repository, and commit your first files

<!-- No answer needed here -- we check the repo directly. -->

### 5: Download the reference transcriptome

<!-- Put the download command in `protocol.md`, not here. -->

Code to count the number of transcripts:

```bash
 zgrep -c '^>' data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz
```

The output of the command was:

```
28624
```

*Answer*: The number of transcripts in the file: 28624

------------------------------------------------------------------------

## Part B: Read trimming

### 6: Run TrimGalore

<!-- Put the code to run TrimGalore in `protocol.md`, not here. -->

*Answer*: Whether all batch jobs succeeded, and how you checked that
(and any problems you encountered, and how you solved them): Yes, all the TrimGalore batch jobs were successful. I checked the ongoing jobs by using squeue -u ghimire75 and after the completion of the job, i checked the slurm log files to verify if they ran successfully or not. I checked for the message ' successfully finished script trimgalore.sh'.

I initially expected the slurm log files to be inside the results/trimgalore/logs because the script created the directory. I moved those files into results/trimgalore/logs/ after confirming that all jobs completed successfully.

------------------------------------------------------------------------

## Part C: Transcript quantification with Salmon

### 7: Create the Salmon index

Code to check the size of the index dir:

```bash
ls -lh results/salmon_index
```

The output of the command was:

```
total 1.1G
-rw-rw----+ 1 ghimire75 PAS3493 5.7K Nov  5 11:28 duplicate_clusters.tsv
-rw-rw----+ 1 ghimire75 PAS3493 3.3M Nov  5 11:29 index.ctab
-rw-rw----+ 1 ghimire75 PAS3493 1.5M Nov  5 11:29 index.ectab
-rw-rw----+ 1 ghimire75 PAS3493 722K Nov  5 11:29 index.refinfo
-rw-rw----+ 1 ghimire75 PAS3493  32M Nov  5 11:29 index.ssi
-rw-rw----+ 1 ghimire75 PAS3493 1.5M Nov  5 11:29 index.ssi.mphf
-rw-rw----+ 1 ghimire75 PAS3493 8.3M Nov  5 11:29 index.tct
-rw-rw----+ 1 ghimire75 PAS3493 951M Nov  5 11:29 index.tdct
-rw-rw----+ 1 ghimire75 PAS3493  888 Nov  5 11:29 info.json
drwxrwx---+ 2 ghimire75 PAS3493 4.0K Nov  5 11:28 logs
-rw-rw----+ 1 ghimire75 PAS3493  86M Nov  5 11:29 refseq.bin
-rw-rw----+ 1 ghimire75 PAS3493 247K Nov  5 11:29 refseq_offsets.json
```

*Answer*: How long it took to create the index, and how you found out: 

It took 27 seconds to create the index. I saw the timestamps in the slurm log file, it says the script started at 11:28:51 am and ended at 11:29:18 am. 

*Answer*: The size of the index dir:   
 1.1G


Note: My first attempt failed because the variables were not set correctly. I then used the full file paths and reran the job successfully.

### 8: Run Salmon for all samples

<!-- No answer needed here -- we check `protocol.md` directly. -->

### 9: The transcripts with the most reads

*Answer*: What the `NumReads` column contains:

An estimate of the number of sequencing reads assigned to each transcript by Salmon during quantification.

```bash
tail -n +2 results/salmon_quant/S63/quant.sf | sort -k5,5nr | head -3
```

The output of the command was:

```
XM_039591534.2  6566    6315.086        14518.656821    24004.889
XM_039573735.1  6445    6194.086        5342.073324     8663.254
XM_039573733.1  6591    6340.086        2876.480244     4774.749
```

### 10: The gene of the top transcript

```bash
zgrep "^>XM_039591534.2" data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz
```

The output of the command was:

```
>XM_039591534.2 PREDICTED: Culex pipiens pallens vitellogenin-A1-like (LOC120426760), mRNA
```

*Answer* The gene that the top transcript belongs to: 
  
  LOC120426760 (vitellogenin-A1-like)

### 11: Mapping rates

```bash
grep "mapping rate" results/salmon_quant/*/logs/salmon_quant.log
```

The output of the command was:

```
results/salmon_quant/S63/logs/salmon_quant.log:mapping rate: 82.4560%
results/salmon_quant/S64/logs/salmon_quant.log:mapping rate: 81.9152%
results/salmon_quant/S65/logs/salmon_quant.log:mapping rate: 79.1055%
results/salmon_quant/S66/logs/salmon_quant.log:mapping rate: 81.9350%
results/salmon_quant/S67/logs/salmon_quant.log:mapping rate: 83.0459%
results/salmon_quant/S68/logs/salmon_quant.log:mapping rate: 78.8070%
results/salmon_quant/S69/logs/salmon_quant.log:mapping rate: 82.1824%
```

*Answer*: The sample with the lowest mapping rate: S68 with 78.8070%.

------------------------------------------------------------------------

## Part D: Summarizing all steps with MultiQC

### 12: Update your MultiQC script

<!-- No need to paste the script: we'll look at scripts/multiqc.sh and its Git history directly. -->

### 13: Run MultiQC

*Answer*: For each program, the number of reports that MultiQC found,
and why it found this number:
1.Salmon: 7 meta reports, 7 fragment length distributions and 7 library format counts reports were found. It found 7 of each because there were 7 samples.
2.TrimGalore: 14 reports were found. It found 14 because they are pair ended, so 2 for each samples.
3.FastQC: 14 reports were found. Same as above it was run on pair ended, so 2 for each samples.

### 14: The General Statistics table

*Answer*: The sample with the lowest Salmon mapping rate,
and whether this matches your answer to question 11: 

The sample with the lowest Salmon mapping rate was S68 and it matches with my answer to Q11.

### 15: Adapter lengths

*Answer*: The most common adapter lengths, whether this surprised you,
and why you think these lengths are the most common:

The most common adapter lenghts trimmed were 1-3 bp.
This was not surprising because many reads only contained a very small part of an adapter at the end.
I think these short adapter lengths are the most common because many reads only extend a small distance into the adapter sequence at the end. As a result, Trim Galore only needs to remove a few bp during trimming.

### 16: Ask Claude about the MultiQC report

*Answer*: What you asked, and what you learned from Claude's answer (in your own words):

I asked Claude what the "Overrepresented Sequences" section of the MultiQC report means and whether a warning or failure in this section indicates a problem with my data.

From Claude's explanation, I learned that overrepresented sequences are read sequences that appear much more often than expected. This can happen because of adapter contamination, PCR duplication, rRNA contamination or because some genes are naturally expressed at very high levels in RNA-seq data. I also learned that the "Possible Source" column compares these sequences against a small database of common adapters and primers. A warning or failure in this section does not automatically mean the data are poor quality. For RNA-seq data, highly abundant transcripts can cause overrepresented sequences even when the data are valid. The most important thing is to check whether the sequences are coming from adapters or contamination rather than from real biological sequences.

------------------------------------------------------------------------

## Part E: Improving the workflow with genAI

### 17: Claude's review of your protocol and scripts

*Answer*: At least three of Claude's suggestions,
and whether you agree or disagree with each one, and why:

1.Claude suggested putting all commands inside proper bash code blocks in protocol.md. I agree with this suggestion because it makes the protocol easier to read and helps users clearly tell the difference between commands and regular text. This makes it easier for someone to follow the workflow correctly.

2.Claude suggested adding a checksum for the reference transcriptome file. I agree with this suggestion because it helps confirm that the downloaded file is the correct one and was not damaged during the download process.

3.Claude suggested listing the software and container versions used in the workflow. I agree with this suggestion because it helps future users know exactly which versions were used and makes it easier to reproduce the same results.

### 18: Address a suggestion

*Answer*: Which suggestion(s) you addressed, and what you changed:

I addressed Claude's suggestions to improve the formatting and consistency of protocol.md. I placed the commands for Steps 1–4 inside proper bash code blocks, making it easier to distinguish commands from explanatory text and follow the workflow.

I also addressed Claude's suggestion to document the software versions used in the workflow. I added a Software Versions Used section to the protocol that records the versions of Trim Galore, Salmon and MultiQC used in the analysis.

While making these changes, I noticed that I had defined variables earlier in the protocol but was not consistently using them in later steps. I updated the commands to use the previously defined variables instead of repeating file paths directly. This makes the protocol more consistent, easier to maintain and less likely to contain errors if file locations need to be changed in the future.

------------------------------------------------------------------------

## Part F: Publish your repo on GitHub

### 19: Create a GitHub repository and push

<!-- No answer needed here -- we check your GitHub repo directly. -->

### 20: Get a (test) DOI with Zenodo

*Answer*: Your Zenodo Sandbox DOI: [![DOI](https://sandbox.zenodo.org/badge/1384067995.svg)](https://handle.test.datacite.org/10.5072/zenodo.637499)
