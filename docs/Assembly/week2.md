## Introduction

Today we will start with generating three assemblies: 

- Whole genome assembly using [`flye`](https://github.com/fenderglass/Flye)
- Whole genome assembly using [`canu`](https://github.com/marbl/canu)
- Whole transcriptome assembly using [`Trinity`](https://github.com/trinityrnaseq/trinityrnaseq/wiki)

Note that assemblies are generally compute intense, particularly in terms of memory. So, your jobs will take at least a few hours to complete and require quite some memory.

While `flye` and `Trinity` run within a single job, `canu` creates different job steps for you. This means that for `flye` and `Trinity` you will need to reserve the appropriate CPU and memory by using the `sbatch` options. The `canu` command itself runs very quickly, and it submits different jobs automatically that are needed to complete the canu assembly. 

## flye assembly 

Generate and run a script for performing the `flye` assembly from the pacbio reads that you can submit using `sbatch`. Use (amongst others) the following `sbatch` options:

```sh
#SBATCH --time=06:00:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=16
```

!!! hint 
        Find the usage of `flye` [here](https://github.com/fenderglass/Flye/blob/flye/docs/USAGE.md)

## canu assembly 

Generate and run a script for performing the `canu` assembly from the pacbio reads. Have a look [here](https://canu.readthedocs.io/en/latest/quick-start.html#assembling-pacbio-clr-or-nanopore-data) to get started. 

In addition, you will need some specific options to tell `canu` how to submit to the cluster, these are: 

- **`gridEngineResourceOption`**: the options how `canu` can specify resources within a job. For SLURM this is: `gridEngineResourceOption="--cpus-per-task=THREADS --mem-per-cpu=MEMORY"`
- **`gridOptions`**: options you would otherwise add to the `sbatch` options, e.g.: `gridOptions="--partion=pall --mail-user=mymail@students.unibe.ch"`

More information on grid configuration [here](https://canu.readthedocs.io/en/latest/parameter-reference.html#grid-engine-configuration)

As `canu` creates the jobs for you, you can submit the `canu` command itself with limited resources, e.g.:

```sh 
#SBATCH --time=01:00:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=1
```

## Trinity assembly 

Find information on how to run `Trinity` [here](https://github.com/trinityrnaseq/trinityrnaseq/wiki). You can provide multiple fastq files if they are separated by a comma. 

You can use (amongst others) the following `sbatch` options:

```sh
#SBATCH --time=1-00:00:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=12
```