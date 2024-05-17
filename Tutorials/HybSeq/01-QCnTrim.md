# Quality-control and read trimming

## Run FastQC, dedup, trim, then run FastQC again

To do all the QC steps, we will first create a batch file that will be used to send all the separate jobs to SLURM. The batch file will call for an "array", which enables the same script to be run several times with slight modification. In this case, we will run the same script, with the only modification being the .fastq.gz reads used as input.
```bash
SRC=/project/def-bourret/shared/progs
READS_DIR=/project/def-bourret/shared/test/HybSeqTest/reads
SCRATCH=~/scratch
EMAIL=etienne.leveille-bourret@umontreal.ca
ADAPTER=TruSeq3-PE-2.fa

## create folders for deduplicated and trimmed reads
mkdir -p $READS_DIR/dedup
mkdir -p $READS_DIR/trim

## create slurm batchfile
cd $READS_DIR

echo '#!/bin/bash' > QCnTrim.sbatch
echo "#SBATCH --job-name=readQCnTrim
#SBATCH --output=readQCnTrim-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=2G
#SBATCH --time=0-03:00:00

## FastQC raw reads
cd $READS_DIR
READ1=\$(ls *_1.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _1.fastq.gz)
READ2=\$(echo \"\${NAME}_2.fastq.gz\")

module load java/17.0.6

echo \"Running FastQC on raw \$READ1\"
$SRC/FastQC/fastqc \$READ1
echo \"Running FastQC on raw \$READ2\"
$SRC/FastQC/fastqc \$READ2

## remove PCR duplicates using SuperDeduper from HTStream
hts_SuperDeduper \\
  -1 \$READ1\\
  -2 \$READ2 \\
  -s 5 \\
  -l 85 \
  --stats-file $READS_DIR/dedup/dedup_stats_\$SLURM_ARRAY_TASK_ID.log \\
  -f $READS_DIR/dedup/\${NAME}_dedup
  
## trim adapter seqs and low qual bases
module load trimmomatic/0.39
cd $READS_DIR/trim
java -jar \$EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE -phred33 \\
   $READS_DIR/dedup/\${NAME}_dedup_R1.fastq.gz \\
   $READS_DIR/dedup/\${NAME}_dedup_R2.fastq.gz \\
   \${NAME}_trim_R1.fastq.gz \${NAME}_unpaired_R1.fastq.gz \\
   \${NAME}_trim_R2.fastq.gz \${NAME}_unpaired_R2.fastq.gz \\
   ILLUMINACLIP:/project/def-bourret/shared/progs/Trimmomatic-0.39/adapters/$ADAPTER:2:30:10:8:TRUE \\
   LEADING:3 \\
   TRAILING:3 \\
   SLIDINGWINDOW:4:15 \\
   MINLEN:35
   
## FastQC final (deduplicated and trimmed) reads
echo \"Running FastQC on final (duplicated and trimmed) \${NAME}_trim_R1.fastq.gz\"
$SRC/FastQC/fastqc \${NAME}_trim_R1.fastq.gz
echo \"Running FastQC on final (duplicated and trimmed) \${NAME}_trim_R2.fastq.gz\"
$SRC/FastQC/fastqc \${NAME}_trim_R2.fastq.gz" >> QCnTrim.sbatch

## submit this batchfile to SLURM
NFILES=$(ls -1 *_1.fastq.gz | wc -l)
conda activate htstream
sbatch --mail-user=$EMAIL --array=1-$NFILES QCnTrim.sbatch

## check progress on SLURM queue
sq

```

zcat ../SRR14320985_1.fastq.gz | head -n40000 | gzip > test_1.fastq.gz
zcat ../SRR14320985_2.fastq.gz | head -n40000 | gzip > test_2.fastq.gz
module load bbmap/39.06
clumpify.sh \
  in1=test_1.fastq.gz \
  in2=test_2.fastq.gz \
  out1=test_1.dedup.fastq.gz \
  out2=test_2.dedup.fastq.gz \
  dedupe=t \
  shortname=t \
  unpair=t \
  repair=t \
  tmpdir=$SCRATCH \
  usetmpdir=t \
  -Xmx1g


zcat ../SRR14320985_1.fastq.gz | head -n40000 | gzip > test_1.fastq.gz
zcat ../SRR14320985_2.fastq.gz | head -n40000 | gzip > test_2.fastq.gz
conda activate htstream
hts_SuperDeduper -1 test_1.fastq.gz -2 test_2.fastq.gz -f test/test_dedup



Download FastQC html files on local computer, to check them in browser. Do not forget to change `LOCAL` to a path on your own computer, and then run this code on a **local** terminal. For Windows users, the path of `LOCAL` begins with `/mnt/c` and needs to use only forward slashes (not backward slashes in Unix, they mean something else!):
```bash
LOCAL=~/Downloads
WD=/project/def-bourret/shared/test/HybSeqTest/reads
WD2=/project/def-bourret/shared/test/HybSeqTest/reads/trim

cd $LOCAL
rsync --progress bourret@beluga.computecanada.ca:$WD/*_fastqc* $LOCAL/.
rsync --progress bourret@beluga.computecanada.ca:$WD2/*_fastqc* $LOCAL/.

```

The test dataset contains a large number of PCR or optical duplicates, such that deduplication removes greater than half of the reads in nearly all the samples. However, the read quality is fine, and the final number of deduplicated reads should be enough for downstream analyses.



