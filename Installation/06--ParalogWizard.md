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

Clone the May 21, 2024 Experimental code for ParalogWizard, which includes preliminary ploidy detection using nQuire.
```bash
WD=/project/def-bourret/shared/progs
cd $WD
git clone --depth=1 --branch Experimental_code https://github.com/rufimov/ParalogWizard.git
mv ParalogWizard ParalogWizard_experimental
```



