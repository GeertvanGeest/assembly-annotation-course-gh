## Run containers 

Some software requires very specific dependencies and can not be installed as a module. These kind of software are available in containers. You can find them (like the modules) through the [vital-it website](https://www.vital-it.ch/services). You can run the containers with `singularity`. The syntax is like this: `singularity exec --bind [path to working directory] [path to container] [command]`. You can find more information on the container of interest like this:

```sh
# example for Merqury:
singularity run-help /software/singularity/containers/Merqury-1.3-1.ubuntu20.sif 
```

Running the program `merqury.sh` inside this container would look like:

```sh
WORKDIR=/path/to/work/directory

singularity exec \
--bind $WORKDIR \
/software/singularity/containers/Merqury-1.3-1.ubuntu20.sif \
merqury.sh \
--help
```

Note that `singularity` is not available on the head node (binfservms01). Therefore you can run singularity only within a job on any other node on the cluster. To quickly check the helper you can use:

```sh 
srun singularity run-help /software/singularity/containers/Merqury-1.3-1.ubuntu20.sif
```

To run the container (or any larger process) it's best to submit it as a job with `sbatch`. For this create a script (e.g. `merqury.sh`):

```sh 
#!/usr/bin/env bash

#SBATCH --time=1-00:00:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=4
#SBATCH --job-name=merqury

PROJDIR=/path/to/project/dir

singularity exec \
--bind $PROJDIR \
/software/singularity/containers/Merqury-1.3-1.ubuntu20.sif \
merqury.sh \
$PROJDIR/quality_control/meryl/Ler.meryl \
$PROJDIR/assemblies/flye.fasta \
flye_test 
```

And submit it with:

```sh
sbatch merqury.sh
```

## quast

```sh
#!/usr/bin/env bash

#SBATCH --mail-user=geert.vangeest@bioinformatics.unibe.ch
#SBATCH --mail-type=end,fail
#SBATCH --time=00:05:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=4
#SBATCH --output=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler/log/quast_output_%j.txt
#SBATCH --error=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler/log/quast_error_%j.txt
#SBATCH --job-name=p651_quast
#SBATCH --partition=pall

PROJDIR=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler

mkdir -p $PROJDIR/quality_control/quast

cd $PROJDIR/quality_control/quast

singularity exec \
--bind $PROJDIR \
$PROJDIR/../containers/quast_5.1.0rc1.sif \
quast.py \
--output-dir $PROJDIR/quality_control/quast \
--min-contig 100 \
--threads ${SLURM_CPUS_PER_TASK} \
--eukaryote \
--est-ref-size 135000000 \
--labels flye,canu \
$PROJDIR/assemblies/flye.fasta \
$PROJDIR/assemblies/canu.fasta
```

## Run BUSCO 

```sh 
#!/usr/bin/env bash

#SBATCH --mail-user=geert.vangeest@bioinformatics.unibe.ch
#SBATCH --mail-type=end,fail
#SBATCH --time=1-00:00:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=12
#SBATCH --output=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler/log/busco_flye_output_%j.txt
#SBATCH --error=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler/log/busco_flye_error_%j.txt
#SBATCH --job-name=p651_busco
#SBATCH --partition=pall

# module add UHTS/Analysis/busco/4.1.4

PROJDIR=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler
# export AUGUSTUS_CONFIG_PATH=$PROJDIR/augustus_config

# brassicales_odb10

singularity run \
--bind $PROJDIR \
$PROJDIR/../containers/busco_5.1.3--pyhdfd78af_0.sif \
busco \
-i $PROJDIR/Ler-deNovoAssembly_flye/assembly.fasta \
-l brassicales_odb10 \
-o BUSCO \
-m genome \
--cpu ${SLURM_CPUS_PER_TASK} \
--out_path $PROJDIR/Ler-deNovoAssembly_flye/ \
--download_path $PROJDIR \
--force
```

## merqury 

Look at: https://kat.readthedocs.io/en/latest/walkthrough.html#genome-assembly-analysis-using-k-mer-spectra

```sh
#!/usr/bin/env bash

#SBATCH --mail-user=geert.vangeest@bioinformatics.unibe.ch
#SBATCH --mail-type=end,fail
#SBATCH --time=1-00:00:00
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=4
#SBATCH --output=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler/log/merqury_output_%j.txt
#SBATCH --error=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler/log/merqury_error_%j.txt
#SBATCH --job-name=p651_merqury
#SBATCH --partition=pall

PROJDIR=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler

mkdir -p $PROJDIR/quality_control/meryl
mkdir -p $PROJDIR/quality_control/merqury
mkdir -p $PROJDIR/assemblies

ln -s $PROJDIR/Ler-deNovoAssembly_flye/assembly.fasta $PROJDIR/assemblies/flye.fasta
ln -s $PROJDIR/Ler-deNovoAssembly_canu/Ler.contigs.fasta $PROJDIR/assemblies/canu.fasta

# singularity exec \
# --bind $PROJDIR \
# /software/singularity/containers/Merqury-1.3-1.ubuntu20.sif \
# best_k.sh 135000000


singularity exec \
--bind $PROJDIR \
/software/singularity/containers/Merqury-1.3-1.ubuntu20.sif \
meryl \
k=18 \
count output \
$PROJDIR/quality_control/meryl/Ler.meryl \
$PROJDIR/reads/ERR3624574_1.fastq.gz
$PROJDIR/reads/ERR3624574_2.fastq.gz

cd $PROJDIR/quality_control/merqury/

singularity exec \
--bind $PROJDIR \
/software/singularity/containers/Merqury-1.3-1.ubuntu20.sif \
merqury.sh \
$PROJDIR/quality_control/meryl/Ler.meryl \
$PROJDIR/assemblies/flye.fasta \
flye_test 
```

