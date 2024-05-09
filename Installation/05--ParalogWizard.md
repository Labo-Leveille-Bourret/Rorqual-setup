# Installing ParalogWizard

## Installation

Clone the current ParalogWizard github repository:
```
WD=/project/def-bourret/shared/progs
cd $WD
wget https://github.com/rufimov/ParalogWizard/archive/refs/tags/0.2.tar.gz
tar -xvf 0.2.tar.gz
rm 0.2.tar.gz

```

## Testing with a small Crataegus dataset

Create a small dataset of 4 species to test ParalogWizard, 200,000 reads/sample.
```bash
WD=/project/def-bourret/shared/data/Crataegus_hybseq
cd $WD
mkdir -p smalltest
cd smalltest

READPATH=$WD/10rawreads/Crataegus-populnea_HbPr32
NAME=Cpopulnea
zcat $READPATH/*R1*.fastq.gz | head -n800000 | gzip > ${NAME}_1.fastq.gz
zcat $READPATH/*R2*.fastq.gz | head -n800000 | gzip > ${NAME}_2.fastq.gz

READPATH=$WD/10rawreads/Crataegus-macrosperma_FtPr48
NAME=Cmacrosperma
zcat $READPATH/*R1*.fastq.gz | head -n800000 | gzip > ${NAME}_1.fastq.gz
zcat $READPATH/*R2*.fastq.gz | head -n800000 | gzip > ${NAME}_2.fastq.gz

READPATH=$WD/10rawreads/Crataegus-suborbiculata_FtPr34
NAME=Csuborbiculata
zcat $READPATH/*R1*.fastq.gz | head -n800000 | gzip > ${NAME}_1.fastq.gz
zcat $READPATH/*R2*.fastq.gz | head -n800000 | gzip > ${NAME}_2.fastq.gz

READPATH=$WD/10rawreads/Crataegus-brainerdii_FtPr02
NAME=Cbrainerdii
zcat $READPATH/*R1*.fastq.gz | head -n800000 | gzip > ${NAME}_1.fastq.gz
zcat $READPATH/*R2*.fastq.gz | head -n800000 | gzip > ${NAME}_2.fastq.gz

```

Send Malineae481 probe sequences to cluster.
```bash
LOCAL=~/Downloads/Malinae_Kew_probes*.fas
REMOTE=bourret@beluga.computecanada.ca:/project/def-bourret/shared/data/Crataegus_hybseq/

rsync --progress $LOCAL $REMOTE

```

Try to run the ParalogWizard pipeline on cluster:
```bash
WD=~/scratch/pw_test
RAW_READS=/project/def-bourret/shared/data/Crataegus_hybseq/smalltest
BAITS_CONCAT=/project/def-bourret/shared/data/Crataegus_hybseq/Malinae_Kew_probes_concat_exons_introns_HP.fas
PW_SRC=/project/def-bourret/shared/progs/ParalogWizard-0.2
mkdir -p $WD
cd $WD

## create links to raw reads
mkdir -p reads
cd reads
ln -s $RAW_READS/* .

## create links to bait seqs
cd $WD
ln -s $BAITS_CONCAT baits_concat.fasta

## create sample list
echo "Cpopulnea
Cmacrosperma
Csuborbiculata
Cbrainerdii

" > samples_list.txt

## try running ParalogWizard
python3 $PW_SRC/ParalogWizard.py cast_assemble -d reads -pr baits_concat.fasta

```


