## Using software

On the IBU cluster software is available in modules or containers. More info on how to use modules [here](https://doc.bioinformatics.unibe.ch/cluster_wiki/HPC_tutorial/SLURM_tutorial/#4-modules). If you need software, try to find it at the [vital-it website](https://www.vital-it.ch/services). You can load the software modules with `module add [PATH]`, to make it available:

```sh
module add UHTS/Quality_control/fastqc/0.11.9
fastqc --help
```

## Raw input reads

!!! bug "work to be done here"
    We will probably make the reads available in a central place (and have multiple accessions). 

PacBio reads of Ler are at ERR3415825  and ERR3415826

```sh
module add UHTS/Analysis/sratoolkit/2.10.7

for SRA_ID in ERR3415825 ERR3415826
do
  prefetch $SRA_ID
  fastq-dump \
  --gzip \
  $SRA_ID
done
```

The whole genome Illumina reads of Ler:


```sh
prefetch ERR3624574
fastq-dump \
--gzip \
--defline-seq '@$sn[_$rn]/$ri' \
--split-files ERR3624574 
```

Download also Illumina the paired-end RNA-seq reads of Ler:

```sh
for SRA_ID in SRR5230991 SRR5230992 SRR5230993
do
  prefetch $SRA_ID
  fastq-dump \
  --gzip \
  --defline-seq '@$sn[_$rn]/$ri' \
  --split-files $SRA_ID
done
```



## Basic read statistics

You have the following reads available:

- Whole genome PacBio reads
- Whole genome Illumina reads
- Whole transcriptome Illumina reads 

**Exercise:** Run `fastqc` on each of the fastq files. To do this, first look up the required module at [https://www.vital-it.ch/services](https://www.vital-it.ch/services), and create a script to run `fastqc` using SLURM's `sbatch`. 

!!! note "using SLURM"
    A tutorial about job submission using SLURM on the IBU cluster you can find [here](https://doc.bioinformatics.unibe.ch/cluster_wiki/HPC_tutorial/SLURM_tutorial/). 

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
