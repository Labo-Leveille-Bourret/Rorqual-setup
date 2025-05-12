# Setting up conda, conda environments and programs

## Every user needs to run the code below to be able to access conda!!!

Users in my group will need to run this code to be able to access Miniconda:
```bash
SRC=/project/def-bourret/shared/progs/miniconda/bin
$SRC/conda init bash
$SRC/conda config --set auto_activate_base false

```

## Installing and setting up conda

Users in my group *PLEASE DO NOT RUN THE CODE BELOW*, as these are already installed.

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

## wait for previous command to finish (saying "y" to all) and then run:
rm ./Miniconda3-latest-Linux-x86_64.sh

```

*Before continuing:* run the command in the section above to initialize conda, close the connection with the cluster, and then reconnect.


Then, install some basic python programs in the base environment:
```bash
SRC=/project/def-bourret/shared/progs/miniconda/bin
conda activate base
$SRC/conda config --add channels defaults
$SRC/conda config --add channels bioconda
$SRC/conda config --add channels conda-forge
conda install -c bioconda newick_utils
conda install -c bioconda catfasta2phyml

```

## Installing conda environments and programs

Creating environments and installing software inside those environments:
```bash
SRC=/project/def-bourret/shared/progs/miniconda/bin
WD=/project/def-bourret/shared/progs

## biopython
$SRC/conda create -n bio -c conda-forge biopython matplotlib


## newick_utils
$SRC/conda create -n newick_utils -c bioconda newick_utils

## hybpiper
$SRC/conda create -n hybpiper -c chrisjackson-pellicle hybpiper

## getOrganelle
$SRC/conda create -n getOrganelle -c bioconda getorganelle
conda activate getOrganelle
get_organelle_config.py --config-dir $WD/getOrganelle --add embplant_pt,embplant_mt,embplant_nr

## HTStream
$SRC/conda create -n htstream -c bioconda htstream

## RagTag
$SRC/conda create -n ragtag -c bioconda ragtag

## ipyrad
$SRC/conda create -y -n ipyrad -c conda-forge ipyrad

## wgd
$SRC/conda create -y -n wgd -c bioconda click pandas matplotlib rich bio wgd

## list all environments
$SRC/conda info --envs

```



## Problems to solve

I can't seem to install NovoWrap correctly in a conda environment. Here is what I tried up to now:

## NovoWrap
## Following issue #13 on the github, I found out that I need to install using pip, but still not working
$SRC/conda create -n novowrap python=3.8 pip
conda activate novowrap
$SRC/../envs/novowrap/bin/pip install -U --force-reinstall novowrap==1.30

$SRC/conda create -n novowrap -c wpwupingwp biopython=1.77 novowrap
To remove:
conda install 'wheel>=0.32.3' 'biopython>=1.77' 'coloredlogs>=10.0' 'matplotlib>=3.2.0' 'numpy>=1.19.1' 'pip>=18.0' 'graphviz>=0.13'
