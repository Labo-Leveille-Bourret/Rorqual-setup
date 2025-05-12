# Retrieving non-target plastid, mitochondrial and nuclear ribosomal DNA

All these genomic compartments are repeated and circular (plastid, mitochondrial) or can be represented as circular (tandem nrDNA repeats). Hence, they can be assembled using the same general approach.

We will be using the program [getOrganelle](https://github.com/Kinggerm/GetOrganelle) to assemble these high-copy regions from the non-target HybSeq reads. This program starts by mapping reads to a reference genome (e.g., plastid genome already assembled in a close relative), and it assembles the mapped reads *de novo*. Then, it maps again the raw reads, but this time to the contigs obtained in the previous steps. This generally results in a higher number of mapped reads, because the new reference now comes from the same genome as the reads being mapped. The newly mapped reads are included in a new *de novo* assembly, creating larger and more accurate contigs. Then, raw reads are again mapped to the new contigs, and included in the assembly, etc. By doing these mapping + assembly + remapping steps iteratively many (dozens or hundreds of) times, it is possible to obtain complete or nearly-complete plastid genomes or nrDNA cistrons. The mitochondrial genome of plants is usually harder to assemble completely, because there are too many structural rearrangements and repeats, but the approach can still be tried.

## Assembling the plastid genome

To assemble the plastid genome from all the test samples, we will create a SLURM batch file that includes a getOrganelle analysis, and then run it.
```bash
SRC=/project/def-bourret/shared/progs
READS_DIR=/project/def-bourret/shared/test/HybSeqTest/reads/trim
WD=~/scratch/HybSeqTest/pla
REF=/project/def-bourret/shared/genomeRefs/pla/Dryopteris_sieboldii_plastidGenome_MN623354.fasta
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-03:00:00"

mkdir -p $WD
cd $WD

## SLURM batch file for getOrganelle assembly
echo '#!/bin/bash' > getPlastid.sbatch
echo "#SBATCH --job-name=getPlastid
#SBATCH --output=getPlastid-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

## script begins here ##

READ1=\$(ls $READS_DIR/*_trim_R1.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _trim_R1.fastq.gz)

cd $WD

get_organelle_from_reads.py \\
  -t 8 \\
  -1 $READS_DIR/\${NAME}_trim_R1.fastq.gz \\
  -2 $READS_DIR/\${NAME}_trim_R2.fastq.gz \\
  -o $WD/\$NAME \\
  --max-reads Inf \\
  -R 1000 \\
  -w 50 \\
  -k 17,21,31,37,45,53,61,67,75,83,91,97,113,121,127 \\
  --config-dir $SRC/getOrganelle \\
  -F embplant_pt" >> getPlastid.sbatch

## submit this batchfile to SLURM
NFILES=$(ls -1 $READS_DIR/*trim_R1.fastq.gz | wc -l)
conda activate getOrganelle
sbatch --mail-user=$EMAIL --array=1-$NFILES getPlastid.sbatch


conda activate ragtag
cd $WD
mkdir -p newScaffolds
cd newScaffolds
NAME=$(ls -d ../*/ | head -n 1 | tail -1)
NAME=$(basename $NAME)
ln -s ../$NAME/extended_spades/scaffolds.fasta $NAME.fasta

```

Add to the script:

triple the linear ref (ref+ref+ref) and modify the mapping options of ragtag so that contigs are "mapped to all", and in the end taking the "middle" part as the final linearized genome (but this does not work if there are indels near the end or start of the linearized reference genome --> ideal method would be to find a way to automatically circularize the genome by mapping to doubled ref (ref+ref))

ragtag correct (on reference sequence doubled, make sure to *map everywhere*)
ragtag scaffold (on reference sequence doubled *map everywhere*)
fgap

then map reads using bwa
then samtools consensus (using 90% and enable iupac codes)
try and call N below a certain cutoff coverage? (could be 3?)

then find a program to circularize
perhaps circlator?



## Assembling the nrDNA cistron

To assemble the nrDNA cistron from all the test samples, we will create a SLURM batch file that includes a getOrganelle analysis, and then run it.
```bash
SRC=/project/def-bourret/shared/progs
READS_DIR=/project/def-bourret/shared/test/HybSeqTest/reads/trim
WD=~/scratch/HybSeqTest/nr
REF=/project/def-bourret/shared/genomeRefs/nrDNA/
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-03:00:00"

mkdir -p $WD
cd $WD

## SLURM batch file for getOrganelle assembly
echo '#!/bin/bash' > getnrDNA.sbatch
echo "#SBATCH --job-name=getnrDNA
#SBATCH --output=getnrDNA-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

## script begins here ##

READ1=\$(ls $READS_DIR/*_trim_R1.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _trim_R1.fastq.gz)

cd $WD

get_organelle_from_reads.py \\
  -t 8 \\
  -1 $READS_DIR/\${NAME}_trim_R1.fastq.gz \\
  -2 $READS_DIR/\${NAME}_trim_R2.fastq.gz \\
  -o $WD/\$NAME \\
  --max-reads Inf \\
  -R 1000 \\
  -w 50 \\
  -k 17,21,31,37,45,53,61,67,75,83,91,97,113,121,127 \\
  --config-dir $SRC/getOrganelle \\
  -F embplant_nr" >> getnrDNA.sbatch

## submit this batchfile to SLURM
NFILES=$(ls -1 $READS_DIR/*trim_R1.fastq.gz | wc -l)
conda activate getOrganelle
sbatch --mail-user=$EMAIL --array=1-$NFILES getnrDNA.sbatch

## check progress on SLURM queue
sq

```

