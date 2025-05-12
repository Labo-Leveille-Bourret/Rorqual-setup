# Installing sequence data QC, processing, manipulation and alignment programs

## Programs to retrieve and manipulate sequences

Install SRA-toolkit:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget --output-document sratoolkit.tar.gz https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
tar -vxzf sratoolkit.tar.gz

```

Navigate to the bin folder of the SRA-toolkit, and run `bin/vdb-config -i`. Set "Prefer SRA Lite files" to on: this will result in faster downloading and processing of files, at the cost of vary slightly innacurate read qualities. **VERY IMPORTANT: Disable local file caching!!!**


catfasta2phyml:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
git clone https://github.com/nylander/catfasta2phyml.git
chmod +x catfasta2phyml/catfasta2phyml.pl

```

seqtk:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
git clone https://github.com/lh3/seqtk.git;
cd seqtk
make

```

## Programs to QC, filter and trim unaligned sequences

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


## Programs to align sequences

OMM_Macse for alignment of genes and pseudogenes at the nucleotide level based on codon similarity:
```bash
WD=/project/def-bourret/shared/progs

cd $WD

module load StdEnv/2023
module load apptainer/1.2.4

apptainer remote add --no-login SylabsCloud cloud.sycloud.io
apptainer remote use SylabsCloud
apptainer pull --arch amd64 library://vranwez/default/omm_macse:v12.01

```


## Programs to QC and trim alignments


TAPER, to filter out regions in the sequence of a single terminal that is misaligned compared to the 
other sequences.  
```bash
WD=/project/def-bourret/shared/progs

cd $WD

module purge
module load StdEnv/2023


wget https://github.com/chaoszhang/TAPER/archive/refs/tags/v1.0.0.tar.gz
tar -vxzf v1.0.0.tar.gz
rm v1.0.0.tar.gz

## test
module load julia/1.10.0
julia TAPER-1.0.0/correction_multi.jl --help

```

