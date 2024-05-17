# HybPiper

[HybPiper](https://github.com/mossmatters/HybPiper) is the most popular pipeline for the assembly of HybSeq data. 

## Assembling contigs

To assemble reads from all the test samples, we will create a batch file with `nano hybpiper.sbatch` in `~/scratch/HybSeqTest`:
```bash
SRC=/project/def-bourret/shared/progs
READS_DIR=/project/def-bourret/shared/test/HybSeqTest/reads/trim
WD=~/scratch/HybSeqTest
TARGETS=/project/def-bourret/shared/hybseqRefs/GoFlag_targets.fa
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-03:00:00"

mkdir -p $WD
cd $WD

echo '#!/bin/bash' > hybpiper.sbatch
echo "#SBATCH --job-name=hybpiper
#SBATCH --output=hybpiper-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

mkdir -p $SLURM_TMPDIR/output
cd $SLURM_TMPDIR/output

READ1=$(ls $READS_DIR/*1.trimmed.fastq.gz | head -n $SLURM_ARRAY_TASK_ID | tail -1)
NAME=$(basename $READ1 _1.trimmed.fastq.gz)

hybpiper assemble \
  --cpu 8 \
  --run_intronerate \
  -t_dna $TARGETS \
  -r $READS/${NAME}_*.trimmed.fastq.gz \
  --prefix $NAME \
  --bwa

rsync -r --progress $SLURM_TMPDIR/output/$NAME/ $WD/$NAME

```

Then, submit this batchfile to SLURM. Don't forget to correctly setup the array length and set the correct email address.
```bash
WD=/home/$USER/scratch/HybSeqTest
READS=/project/def-bourret/$USER/HybSeqTest/reads/trim
EMAIL=etienne.leveille-bourret@umontreal.ca

cd $WD

## number of fastq.gz read files to analyze
NFILES=$(ls -1 $READS/*_1.trimmed.fastq.gz | wc -l)

conda activate hybpiper
sbatch --mail-user=$EMAIL --array=1-$NFILES hybpiper.sbatch

```

I chose to give only 3h for each of the array jobs to run, because this maximizes the number of nodes on which the jobs can run, hence they tend to run much faster. Follow this link for scheduling policies at Compute Canada: https://docs.alliancecan.ca/wiki/Job_scheduling_policies

### QC of the assembly

The command `hybpiper stats` will give some summary statistics on the HybPiper assembly. It requires us to specify a list of probe sequences, and a list of sample names. The command `hybpiper recovery_heatmap` takes the result of `hybpiper stats` and generates a heatmap to visually assess how well each gene was recovered in each sample (in terms of the length of the longest contig).

Here is a script to generate statistics using `hybpiper stats` and a recovery heatmap with `hybpiper recovery_heatmap`:
```bash
WD=/home/$USER/scratch/HybSeqTest
TARGETS=/project/def-bourret/shared/data/HybSeqRefs/AncPhy_targets.fa

cd $WD

## create a list of samples assembled with hybpiper
ls -d */ | cut -f1 -d'/' > samplelist.txt

conda activate hybpiper

## run hybpiper stats
hybpiper stats -t_dna $TARGETS gene samplelist.txt
cat seq_lengths.tsv

## create a recovery heatmap
hybpiper recovery_heatmap --heatmap_dpi 300 --heatmap_filetype pdf seq_lengths.tsv

```

### Investigating problems of paralogy

Sometimes, one of the genes targeted by the HybSeq probes is duplicated in one or a few of your samples. If the age of the duplication is young enough, it is quite likely that both copies of the gene are similar enough to each other and the probe sequence that they will both be "fished" by the probes. The reads of both copies are then caught by HybPiper and assembled in different contigs (because they are not identical).

By default, HybPiper identifies the "best" contig for each gene by coverage (10x more coverage than other contigs) and length (>=75% of the target sequence is present). If multiple contigs have high coverage and length, they likely each represent a different copy of a duplicated gene (i.e., they are paralogs), or different alleles of the same gene.

To check how many potential paralogs/alternative alleles are present in each sample, run the script below:
```bash
WD=/home/$USER/scratch/HybSeqTest
TARGETS=/project/def-bourret/shared/data/HybSeqRefs/AncPhy_targets.fa

cd $WD

conda activate hybpiper

## create a report on the number of potential paralogs
## run in the background because it takes a bit of time
nohup hybpiper paralog_retriever samplelist.txt -t_dna $TARGETS > paralog_retriever.log &
tail -f paralog_retriever.log

```

Once done, download the file `paralog_report.tsv` to your local computer, and examine it in excel.
- How many loci are single-copy across all samples?
- How many loci possibly contain paralogs?
- Are there any loci with paralogs in every sample, suggesting an ancient gene duplication shared by all species?

HybPiper selects for each locus one contig that it considers the "best representative" for the contig, based on coverage, length and similarity to the probe sequence. However, it is best to manually check all the gene trees to make sure that HybPiper has not made any error.

For each locus, align all the contigs retrieved by paralog retriever using [MAFFT](https://mafft.cbrc.jp/alignment/software/about.html) and generate a quick gene phylogeny using [FastTree](http://www.microbesonline.org/fasttree/):
```bash
WD=/home/$USER/scratch/HybSeqTest

mkdir -p $WD/trees/paralogs
cd $WD/trees/paralogs

module load mafft/7.471
module load fasttree/2.1.11

## loop over all loci, align with mafft, pipe to FastTree for quick rough phylogeny
echo "for i in \$(ls -d $WD/paralogs_all/*.fasta)
  do
    GENENAME=\$(basename \$i)
    mafft --localpair \$i | FastTree -nt -gtr > \$GENENAME.tre
  done" > phylo_loop.sh

## run the loop in the background, save screen output to log file
nohup bash phylo_loop.sh > phylo_loop.log &

```

Once done, download all these tree files to your local computer and go through them using [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) or another tree viewer, focusing in particular on loci that have many paralogs identified for many samples. 
- Is there evidence that allelic contigs were retrieved? 
- Is there evidence that paralogs were retrieved?


