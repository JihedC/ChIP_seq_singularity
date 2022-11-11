# ChIP-seq Snakemake
A snakemake pipeline for the analysis of SE and PE ChIP-seq data using singularity and docker.

[![Snakemake](https://img.shields.io/badge/snakemake-≥5.2.0-brightgreen.svg)](https://snakemake.bitbucket.io)
[![Miniconda](https://img.shields.io/badge/miniconda-blue.svg)](https://conda.io/miniconda)

# Aim
Snakemake pipeline made for reproducible analysis of paired-end Illumina ChIP-seq data. The desired output of this pipeline are:
- fastqc zip and html files
- bigWig files (including bamCompare rule)
- bed files

# How to use the Snakemake pipeline -- TLDR

This paragraph should get the Snakemake workflow to work in few minutes:

- download the pipeline: `git clone https://github.com/JihedC/ChIP_seq_singularity.git`
- change directory to the newly downloaded pipeline: `cd ChIP_seq_singularity/`
- if conda is not installed:
  - download miniconda3: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh`
  - install miniconda in your folder: `sh Miniconda3-latest-Linux-x86_64.sh`
  Say yes for the licence terms and either ENTER to confirm the location of installation or choose another location for the installation.
  After the installation you will need to restart your terminal and reconnect to the HPC.
  At the restart, "(base)" should appear on the left side of the prompt which means that you are in the base environment
- install Mamba: `conda install -n base -c conda-forge mamba`  this might take a while, but it's worth it because it will make conda much faster. Accept all the install.
- activate the base environment: `conda activate base`
- install snakemake with mamba: `mamba create -c conda-forge -c bioconda -n snakemake snakemake` Accept the insllation with "Y"
- activate the snakemake environment: `conda activate snakemake`
- control that you are in the folder containing the workflow/pipeline: `pwd`
- Use the dry run option to check that the download pipeline should work: `snakemake -np` if nothing appears in red, the pipeline should work.
- Adapt the units.tsv file to your sample name and their path on the HPC. Make sure that the columns are tab seaparated values.
- Use the dry run option to check that the pipeline will run now that the units.tsv is adapted: `snakemake -np` if nothing appears in red, the pipeline should work.
- Start the pipeline: `sbatch slurm_snakemake.sh`

Snakemake makes uses of **singularity** to pull images of Dockers containers. Dockers containers contains the softwares required for the rules set up in the Snakemake workflow.
**Singularity is a must and will most likely be the source of error** 
For now I have hard coded the module loaded by Shark: `module load container/singularity/3.10.0/gcc.8.5.0`. If in the future, this module is removed from Shark or modified, it might prevent the pipeline from working because it will not be able to pull containers. This line of code would then need to be modified in the file `slurm_snakemake.sh`.
This line would need to be replaced by the line obtained after running `module spider singularity` on Shark.


# Content of the repository

- **Snakefile** containing the targeted output and the rules to generate them from the input files.

- **config/** , folder containing the configuration files making the Snakefile adaptable to any input files, genome and parameter for the rules. Adapt the config file and its reference in the Snakefile. Please also pay attention to the parameters selected for deeptools, for convenience and faster test the **bins** have been defined at `1000bp`, do not forget to adapt it to your analysis.

- **Fastq/**, folder containing subsetted paired-end fastq files used to test locally the pipeline. Generated using [Seqtk](https://github.com/lh3/seqtk): `seqtk sample -s100 read1.fq 5000 > sub1.fqseqtk sample -s100 read2.fq 5000 > sub2.fq`. RAW fastq or fastq.gz files should be placed here before running the pipeline.

- **units.tsv**, is a tab separated value files containing information about the experiment name, the condition of the experiment (control or treatment) and the path to the fastq files relative to the **Snakefile**. **Change this file according to your samples.**

- **rules/**, folder containing the rules called by the snakefile to run the pipeline, this improves the clarity of the Snakefile and might help modifying the file in the future.


# Usage

## Conda environment

The only conda environment required to run this workflow is the **(snakemake)** environment. Details on how to install this environment can be found in the TLDR section of this documentation. 
I strongly advise to take your time to install mamba since it is much faster than conda. Mamba replaces conda in almost all the usual conda commands. For example:
Instead of using `conda install -c bioconda macs2` one can use `mamba install -c bioconda macs2`.

The versions of the softwares are hard-coded in the rules. If one desires to use a particular version of a software, they would need to find a docker hub containing the desired version and to change it in every rules that uses the software.

## Configuration file

The `config.yaml` file specifies the sample list, the genomic reference fasta file to use, the directories to use, etc. This file is then used to build parameters in the main `Snakefile`.

## Snakemake execution

The Snakemake pipeline/workflow management system reads a master file (often called `Snakefile`) to list the steps to be executed and defining their order.
It has many rich features. Read more [here](https://snakemake.readthedocs.io/en/stable/).

## Samples

Samples are listed in the `units.tsv` file and will be used by the Snakefile automatically. Change the name, the conditions accordingly.

## Dry run

Use the command `snakemake -np` to perform a dry run that prints out the rules and commands.

## Real run

If the dry run is successful (no red lines on the screen), the workflow can be started with `sbatch slurm_snakemake.sh`

# Main outputs

The main output are :

The main output are :

- **fastp** : Provide informations about the quality of the sequences provided and generate a html file to visualize it. More information to be found [here](https://github.com/OpenGene/fastp).Fastp provides 1 files per sample. MultiQC gather all samples together in an html file.

- **bed** : Provide information generated by the MACS2 algorithm for the locations and significance of peaks. These files can be used for direct visualization of the peaks location using IGV or as an input for further analysis using the [bedtools](https://bedtools.readthedocs.io/en/latest/content/bedtools-suite.html)

- **bigwig files** : Provides files allowing fast displays of read coverages track on any type of genome browsers. The default normalization is RPKM and can be modified in the `config.yaml` file.