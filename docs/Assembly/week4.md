## Run nucmer and mummer

```sh 
#!/usr/bin/env bash

module add UHTS/Analysis/mummer/4.0.0beta1

PROJDIR=/data/projects/p651_Assembly_and_annotation_course_2021/assembly_Ler
REFERENCE=$PROJDIR/assemblies/Arabidopsis_thaliana.TAIR10.dna.toplevel.fa
FLYE=$PROJDIR/assemblies/flye.fasta
CANU=$PROJDIR/assemblies/canu.fasta

mkdir -p $PROJDIR/quality_control/mummer/

## gnu plot installed with conda
## conda install -c conda-forge gnuplot

for GENOME in $FLYE $CANU
do
    BASE=`basename $GENOME | cut -f 1 -d '.'`

    nucmer \
    --prefix=$PROJDIR/quality_control/mummer/Ler_${BASE} \
    --breaklen 1000 \
    --mincluster 1000 \
    $REFERENCE \
    $GENOME 
    
    mummerplot \
    $PROJDIR/quality_control/mummer/Ler_${BASE}.delta \
    -R $REFERENCE \
    -Q $GENOME \
    --filter \
    -p $PROJDIR/quality_control/mummer/Ler_${BASE}_plot \
    -t png \
    --fat \
    --large \
    --layout
done

```