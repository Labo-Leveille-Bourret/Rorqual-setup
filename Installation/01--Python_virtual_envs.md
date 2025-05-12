# Creating virtual environments for shared python packages

## Creating virtual environments and installing packages

Creating environments and installing software inside those environments:

Environment for ipyrad:  
```bash
SRC=/project/def-bourret/shared/progs

cd $SRC

module purge
module load StdEnv/2020 python/3.10 muscle/3.8.1551 samtools/1.17 bedtools/2.30.0 vsearch/2.21.1 bwa/0.7.17
virtualenv --no-download $SRC/ipyrad
source $SRC/ipyrad/bin/activate
pip install --no-index --upgrade pip
pip install --no-index ipyrad==0.9.93
python -c "import ipyrad"

```


Environment for Biopython and other basic modules:  
```bash
SRC=/project/def-bourret/shared/progs

cd $SRC

module purge
module load StdEnv/2020 python/3.10
virtualenv --no-download $SRC/biopython
source $SRC/biopython/bin/activate
pip install --no-index --upgrade pip
pip install --no-index biopython
pip install --no-index numpy
pip install --no-index scipy
pip install --no-index pandas

```
