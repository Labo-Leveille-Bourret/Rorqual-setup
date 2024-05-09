# Setting up conda, conda environments and programs

## Every user needs to run the code below to be able to access conda!!!

Users in my group will need to run this code to be able to access Miniconda:
```bash
Miniconda:
```bash
SRC=/project/def-bourret/shared/progs/miniconda/bin
$SRC/conda init bash
$SRC/conda config --set auto_activate_base false

```

## Installing and setting up conda

Install Miniconda:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod +x ./Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh -b -p $WD/miniconda
$WD/miniconda/bin/conda init bash
$WD/miniconda/bin/conda config --set auto_activate_base false
$WD/miniconda/bin/conda update conda
rm ./Miniconda3-latest-Linux-x86_64.sh

```

Install some basic python programs in the base environment:
```bash
conda activate base
conda install -c bioconda newick_utils
conda install -c bioconda catfasta2phyml

```

## Installing conda environments and programs



HybPiper (in conda environment):
```bash
SRC=/project/def-bourret/shared/progs/miniconda/bin
$SRC/conda config --add channels defaults
$SRC/conda config --add channels bioconda
$SRC/conda config --add channels conda-forge
$SRC/conda create -n hybpiper -c chrisjackson-pellicle hybpiper

```

getOrganelle (in conda environment):
```bash

$WD/miniconda/bin/conda create -n getOrganelle -c bioconda getorganelle
conda activate getOrganelle
get_organelle_config.py --config-dir $WD/getOrganelle --add embplant_pt,embplant_mt,embplant_nr

```

