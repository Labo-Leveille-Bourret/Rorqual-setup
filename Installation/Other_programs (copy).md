# Installing required programs

I will here keep notes on how I installed the various programs that are shared within the lab.

## Create shared folder for all members of my group on Béluga

See this webpage for details on how to share data between users: https://docs.alliancecan.ca/wiki/Sharing_data

The problem is that, by default, group members do not have access to files produced by other group members. It is possible to change a folder so that it is accessible by everyone in my group with `chmod g+rwx fichier`, but when a user creates a file in this folder, the file is only accessible to them and not to other users. The user would then have to `chmod` each new file it produces to give access to other group members, which is far from ideal.

Instead, it is possible to create a folder that is accessible to everyone in the group, and inside which files are open to everyone by default (r/w/x):
```bash
cd /project/def-bourret/
mkdir shared
chmod -R g+rwx shared
setfacl -R -d -m g::rwx shared
getfacl shared

```

## Install programs required for the course

Install SRA-toolkit:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget --output-document sratoolkit.tar.gz https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
tar -vxzf sratoolkit.tar.gz

```

Navigate to the bin folder of the SRA-toolkit, and run `bin/vdb-config -i`. Set "Prefer SRA Lite files" to on: this will result in faster downloading and processing of files, at the cost of vary slightly innacurate read qualities. **VERY IMPORTANT: Disable local file caching!!!**

FastQC:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip
unzip fastqc_v0.11.9.zip
rm fastqc_v0.11.9.zip
chmod +x FastQC/fastqc

```

Trimmomatic:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.39.zip
unzip Trimmomatic-0.39.zip
rm Trimmomatic-0.39.zip
chmod +x FastQC/fastqc

```

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

Users in my group will need to run this code to be able to access Miniconda:
```bash
Miniconda:
```bash
SRC=/project/def-bourret/shared/progs/miniconda/bin
$SRC/conda init bash
$SRC/conda config --set auto_activate_base false

```

Install some basic python programs in the base environment:
```bash
conda activate base
conda install -c bioconda newick_utils

```

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

catfasta2phyml:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
git clone https://github.com/nylander/catfasta2phyml.git
chmod +x catfasta2phyml.pl

```

treePL:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
wget https://github.com/blackrim/treePL/archive/refs/heads/master.zip
unzip master.zip
rm master.zip
mv treePL-master treePL
cd treePL/src
module load StdEnv/2020 nlopt/2.7.1 adol-c/2.7.2
./configure
make

```

ASTRAL:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
wget https://github.com/smirarab/ASTRAL/raw/master/Astral.5.7.8.zip
unzip Astral.5.7.8.zip
rm Astral.5.7.8.zip

```


