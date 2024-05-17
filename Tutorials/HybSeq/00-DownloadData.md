# Downloading public HybSeq data

## Tutorial on the use of SRA

### How to find data on SRA

Genbank's [Short Read Archive (SRA)](https://www.ncbi.nlm.nih.gov/sra) is the most important repository of Illumina datasets. It is currently being used to archive raw data produced in nearly all phylogenomic studies, including all the studies we will mine for this course. Here is a short step-by-step guide to obtain data from [Tiley et al. (2024)](https://doi.org/10.1093/sysbio/syae024) from SRA:

1. Search through Tiley et al. (2024) for a SRA study number. At the end of the paper, they mention that the study number is in the supplementary table S1.  
2. Go to [https://www.ncbi.nlm.nih.gov/sra/](https://www.ncbi.nlm.nih.gov/sra/)  
3. Search terms: `SRP316248[study]`, which is the SRA study number  
4. Click Send to -> File -> Accession list  
5. Copy the accession list to `/home/bourret/projects/def-bourret/shared/test/HybSeqTest/SraAccList.csv`  
6. Click Send to -> File -> RunInfo  
7. Copy the metadata to `/home/bourret/projects/def-bourret/shared/test/HybSeqTest/SraRunInfo.csv`

The code below is an example of how this could be transferred (but alternative methods exist):
```bash
## run this on local machine to send files to remote cluster
LOCAL=~/Downloads
REMOTE=/home/bourret/projects/def-bourret/shared/test/HybSeqTest

cd $LOCAL
rsync --progress SraAccList.csv bourret@beluga.computecanada.ca:$REMOTE/
rsync --progress SraRunInfo.csv bourret@beluga.computecanada.ca:$REMOTE/

```

### Code to create a small HybSeq test dataset

The code below fetches Illumina read files for 7 accessions used in Tiley et al. (2024), including two allotetraploids involving some of the other species:

```bash
SRA_SRC=/project/def-bourret/shared/progs/sratoolkit.3.1.0-ubuntu64/bin
WD=/home/bourret/projects/def-bourret/shared/test/HybSeqTest
SCRATCH=/home/$USER/scratch

cd $WD

mkdir -p reads

TARGETS=("Dryopteris campyloptera" "Dryopteris celsa" "Dryopteris expansa" "Dryopteris goldieana" "Dryopteris intermedia" "Dryopteris ludoviciana" "Polystichum munitum")
rm AccList.test
touch AccList.test
for i in "${TARGETS[@]}"
  do
     echo "Getting reads for $i"
     Acc_i=$(grep "$i" SraRunInfo.csv | 
       head -1 | 
       grep -o '^SRR[0-9]*' | 
       head -1)
     echo "$Acc_i"
     echo $Acc_i >> AccList.test
  done

## prefetch the data
nohup $SRA_SRC/prefetch --option-file AccList.test > prefetch.log &

## wait until done

## when done, dump to fastq files
nohup $SRA_SRC/fasterq-dump -t $SCRATCH --split-files SRR* > fasterq-dump.log &

## check progress on task
top -u $USER

## wait until done

## when done, compress the fastq files and delete the prefetch folders
nohup gzip *.fastq &

## wait until done

## when done, cleanup
mv *.fastq.gz ./reads/
rm -r SRR*/ nohup.out

```


