
## Material

**Lecture 1: introduction to genome assembly**

[:fontawesome-solid-file-pdf: Download the presentation](../assets/pdf/general_introduction.pdf){: .md-button }

**Lecture 2: generating sequencing reads**

[:fontawesome-solid-file-pdf: Download the presentation](../assets/pdf/general_introduction.pdf){: .md-button }


## Raw input reads

During this course we will work with raw data from:

Jiao WB, Schneeberger K. **Chromosome-level assemblies of multiple Arabidopsis genomes reveal hotspots of rearrangements with altered evolutionary dynamics.** Nature Communications. 2020;11:1–10. Available from: [http://dx.doi.org/10.1038/s41467-020-14779-y](http://dx.doi.org/10.1038/s41467-020-14779-y)

You can find the input reads at `/data/courses/assembly-annotation-course`. Each group gets appointed a separate accession (i.e. genotype) and each person gets assigned a separate dataset within an accession. 

Find the directory with your dataset like this:

```
/data/courses/assembly-annotation-course/[ACCESSION]/[DATASET]
```

The directory contains the following subdirectories + files (this is an example for An-1):

```
.
├── Illumina
│   ├── ERR3624579_1.fastq.gz
│   └── ERR3624579_2.fastq.gz
├── pacbio
│   ├── ERR3415817.fastq.gz
│   └── ERR3415818.fastq.gz
└── RNAseq
    ├── ERR754061_1.fastq.gz
    └── ERR754061_2.fastq.gz
```

Containing three sequencing datasets: 

- Whole genome Illumina (`Illumina`) 
- Whole genome PacBio (`pacbio`) 
- Whole transcriptome Illumina RNA-seq (`RNAseq`)

Generate a working directory for this course in your data directory at `/data/users/[USERNAME]/assembly_annotation_course`. After that, generate a soft link to the directory with reads to get easy access:

```sh
cd /data/users/[USERNAME]/assembly_annotation_course
ln -s /data/courses/assembly-annotation-course/[ACCESSION]/[DATASET]/ ./
```

## Basic read statistics

In this first exercise you will get acquainted with the different datasets and run some basic QC on them. 

**Exercise:** Run `fastqc` on each of the fastq files. To do this, first look up the required module at [https://www.vital-it.ch/services](https://www.vital-it.ch/services), and create a script to run `fastqc` using SLURM's `sbatch`. 

!!! note "using SLURM"
    A tutorial about job submission using SLURM on the IBU cluster you can find [here](https://doc.bioinformatics.unibe.ch/cluster_wiki/HPC_tutorial/SLURM_tutorial/). 

!!! hint "Work structured!"
    You will use the same dataset throughout this course. Since your building further on your results it is important to keep track of the commands you have run. Therefore, it makes sense to have a directory `scripts` in your working directory (i.e. `/data/users/[USERNAME]/assembly_annotation_course`). In addition, it also helps to keep files related to the same step in the process in the same directory. A directory structure of your working directory could therefore look like this:

    ```
    .
    ├── assemblies
    │   ├── canu
    │   ├── flye
    │   └── Trinity
    ├── read_QC
    │   ├── fastqc
    │   └── kmer_counting
    └── scripts
        ├── 01_read_statistics.sh
        └── 02_kmer_counting.sh
    ```

**Questions:**

- What are the read lengths of the different datasets?
- What kind of coverage do you expect from the Pacbio and the Illumina WGS reads? (**hint**: lookup the expected genome size of *Arabidopsis thaliana*)
- Do all datasets have information on base quality?


## Perform k-mer counting 

**Exercise:** Go to [http://qb.cshl.edu/genomescope/](http://qb.cshl.edu/genomescope/), and follow their suggestions on counting k-mers (`jellyfish count`) and creating a histogram (`jellyfish histo`). Other than what is suggested on the genomescope website, you can use the options `-s 5G` (hash of 5Gb) and `-t 4` (4 threads). The other options you can keep the same as suggested on the website. Again, submit the commands from within a script using `sbatch`.

!!! tip "`jellyfish count` only accepts uncompressed input"
    Your fastq files are compressed (i.e. ending with `fastq.gz`). `jellyfish` can not read those directly, but you can provide the files with process substitution of `zcat`, i.e.:

    ```sh 
    jellyfish count \
    [OPTIONS] \
    <(zcat myreads.fastq.gz) \
    <(zcat myotherreads.fastq.gz)
    ```

**Questions:**

Upload your histo file to the website, checkout the plots and answer the following questions:

- Why are we using canonical k-mers?
- Is the estimated genome size expected? 
- Is the percentage of heterozygousity expected?
