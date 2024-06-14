# Assembling with ipyrad

The test data that will be used for this tutorial is currently at this location: `/project/def-bourret/shared/data/3rad_2024-06-04_plate1/`. It is the first 3RAD library produced at the Centre sur la biodiversité by Simon Joly and Étienne Léveillé-Bourret, and contains samples from various projects in many different taxonomic groups including hawthorns, Bidens and Carex, as well as samples for Simon Joly's research.

Because the dataset contains a combination of samples from different projects, the ipyrad pipeline will be run in 2 separate steps:
1. Demultiplex (=sort) and filter all samples
2. Branch the workflow to select only hawthorn samples, and continue with the rest of the pipeline.

## Demultiplex (=sort), trim and filter

Use ipyrad to demultiplex, trim and filter raw reads:
```bash
SRC=/project/def-bourret/shared/progs
READS=/home/bourret/projects/def-bourret/shared/data/3rad_2024-06-04_plate1/3RAD_test*/*.fq.gz
BARCODES=/home/bourret/projects/def-bourret/shared/data/3rad_2024-06-04_plate1/ipyrad_barcodes.txt
OVERHANG=CTAGA,CTAGC,AATTC
WD=/home/$USER/scratch/RADtest
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-03:00:00"
CORES=32

## create working directory and navigate to it
mkdir -p $WD
cd $WD

## create a subdirectory with symlinks to the read files
mkdir rawreads
ln $READS -s rawreads/
## make sure the names include _R1_ and _R2_ required by ipyrad
rename _1.fq.gz _R1_.fq.gz rawreads/*
rename _2.fq.gz _R2_.fq.gz rawreads/*

## create a ipyrad parameters file
module purge
module load StdEnv/2020 python/3.10 muscle/3.8.1551 samtools/1.17 bedtools/2.30.0 vsearch/2.21.1 bwa/0.7.17
source $SRC/ipyrad/bin/activate
ipyrad -n test

## add path to read files
sed -i "s,                               ## \[2\],$WD/rawreads/* ## \[2\],g" params-test.txt

## add path to barcodes file
sed -i "s,                               ## \[3\],$BARCODES ## \[3\],g" params-test.txt

## change data type to 3RAD
sed -i "s,rad                            ## \[7\],pair3rad                       ## \[7\],g" params-test.txt

## set proper enzyme overhang
sed -i "s/TGCAG,/$OVERHANG,/g" params-test.txt

## allow 1 mismatch in barcode
sed -i "s,0                              ## \[15\],1                              ## \[15\],g" params-test.txt

## allow more heterozygotes in individual consensus seq since hybrids are possible
sed -i "s,0.05                           ## \[20\],0.15                           ## \[20\],g" params-test.txt

## create SLURM batchfile to run demultiplexing + trimming/filtering
echo '#!/bin/bash' > ipyrad_demult.sbatch
echo "#SBATCH --job-name=ipyrad_demult
#SBATCH --output=ipyrad_demult.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

ipyrad -c $CORES -p params-test.txt -s 12" >> ipyrad_demult.sbatch

## submit this batchfile to SLURM
sbatch --mail-user=$EMAIL ipyrad_demult.sbatch

```

## Assemble Carex data

Wait until the previous steps (ipyrad steps 1 and 2) finish. Once they are done, assemble the Carex data, which comprises only diploids, including a few diploid hybrids:
```bash
SRC=/project/def-bourret/shared/progs
WD=/home/$USER/scratch/RADtest
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-24:00:00"
CORES=32
BRANCH=carex1
SAMPLES="Carex_aff_louisianica__ELB_0145 Carex_gigantea_ELB_0267 Carex_gigantea_Florida_Clade_ELB_0229 Carex_louisianica_ELB_0266 Carex_lupuliformis_ish_ELB_0114 Carex_lupuliformis_ELB_0262 Carex_lupuliformis_Quebec_form_ELB_0247 Carex_lupuliformis_Quebec_form_ELB_0248 Carex_lupuliformis_Quebec_form_ELB_0270 Carex_lupuliformish_ELB_0175 Carex_lupuliformish_ELB_0249 Carex_lupulina_ach_larges_ELB_0244 Carex_lupulina_bella-villae_ELB_0242 Carex_lupulina_ELB_0258 Carex_lupulina_x_retrorsa_ELB_0264 Carex_lurida_ELB_0102 Carex_retrorsa_ELB_0243 Carex_retrorsa_ELB_0259 Carex_vesicaria_ELB_0246 Carex_x_cayouettei_ELB_0228"

cd $WD

## create a branch for this assembly
module purge
module load StdEnv/2020 python/3.10 muscle/3.8.1551 samtools/1.17 bedtools/2.30.0 vsearch/2.21.1 bwa/0.7.17
source $SRC/ipyrad/bin/activate
ipyrad -p params-test.txt -b $BRANCH $SAMPLES

## create SLURM batchfile to finish assembly on this branch
echo '#!/bin/bash' > ipyrad_$BRANCH.sbatch
echo "#SBATCH --job-name=ipyrad_$BRANCH
#SBATCH --output=ipyrad_$BRANCH.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

ipyrad -c $CORES -p params-$BRANCH.txt -s 34567" >> ipyrad_$BRANCH.sbatch

## submit this batchfile to SLURM
sbatch --mail-user=$EMAIL ipyrad_$BRANCH.sbatch

```


## Assemble Crataegus data

Wait until the previous steps (ipyrad steps 1 and 2) finish. Once they are done, assemble the Crataegus data, which comprises both diploids and polyploids (probably allopolyploids). Therefore, we also need to adjust the parameters to allow 4 alleles per sample:
```bash
SRC=/project/def-bourret/shared/progs
WD=/home/$USER/scratch/RADtest
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-24:00:00"
CORES=32
BRANCH=crat1
SAMPLES="Crataegus_brainerdii_ELB_0040 Crataegus_canadensis_ELB_0024 Crataegus_chrysocarpa_var_chrysocarpa_ELB_0026 Crataegus_chrysocarpa_var_phoeniceoides_ELB_0025 Crataegus_coccinioides_ELB_0022 Crataegus_crus-galli_ELB_0053 Crataegus_flabellata_ELB_0031 Crataegus_fluviatilis_ELB_0042 Crataegus_fretalis_ELB_0034 Crataegus_grayana_ELB_0030 Crataegus_holmesiana_ELB_0027 Crataegus_irrasa_ELB_0033 Crataegus_macracantha_ELB_0060 Crataegus_punctata_ELB_0058 Crataegus_scabrida_ELB_0049 Crataegus_submollis_ELB_0023 Crataegus_suborbiculata_ELB_0028 Crataegus_x_integriloba_ELB_0029"

cd $WD

## create a branch for this assembly
module purge
module load StdEnv/2020 python/3.10 muscle/3.8.1551 samtools/1.17 bedtools/2.30.0 vsearch/2.21.1 bwa/0.7.17
source $SRC/ipyrad/bin/activate
ipyrad -p params-test.txt -b $BRANCH $SAMPLES

## allow up to 4 alleles per sample
sed -i "s,2                              ## \[18\],4                              ## \[18\],g" params-$BRANCH.txt

## allow most samples to share the same heterozygous sites
sed -i "s,0.5                            ## \[24\],0.9                            ## \[24\],g" params-$BRANCH.txt

## create SLURM batchfile to finish assembly on this branch
echo '#!/bin/bash' > ipyrad_$BRANCH.sbatch
echo "#SBATCH --job-name=ipyrad_$BRANCH
#SBATCH --output=ipyrad_$BRANCH.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

ipyrad -c $CORES -p params-$BRANCH.txt -s 34567" >> ipyrad_$BRANCH.sbatch

## submit this batchfile to SLURM
sbatch --mail-user=$EMAIL ipyrad_$BRANCH.sbatch

```


## Results

Results can be download on your local machine for analysis. Run this on your local machine:
```bash
LOCAL=~/Downloads
WD=/home/bourret/scratch/RADtest

cd $LOCAL
rsync --progress bourret@beluga.computecanada.ca:$WD/*outfiles/*stats.txt $LOCAL/
rsync --progress bourret@beluga.computecanada.ca:$WD/*outfiles/*.snps $LOCAL/

```
