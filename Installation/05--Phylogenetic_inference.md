# Installing phylogenetic inference programs


ASTRAL:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
wget https://github.com/smirarab/ASTRAL/raw/master/Astral.5.7.8.zip
unzip Astral.5.7.8.zip
rm Astral.5.7.8.zip

```

PAUP*:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
wget https://phylosolutions.com/paup-test/paup4a168_centos64.gz
gunzip paup*.gz
chmod +x ./paup*

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

