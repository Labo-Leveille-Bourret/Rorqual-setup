# Convert alignment with IUPAC ambiguity codes to allelic data with heterozygotes

The motivation behind this code is that frequently, heterozygosity in sequence data is represented as IUPAC ambiguity codes in an alignment of DNA sequences. When trying to analyze this using `ade4` and `adegenet`, the most popular `R` packages for population genetics, importing the data is very difficult because they transform all ambiguity codes to "NA", in other words they remove all heterozygous positions from the dataset.

Therefore, it is necessary to import the data using `seqinr` and do a "home-made" conversion from the IUPAC codes to a format that can be read properly by `ade4` and `adegenet`. Only then is it possible to properly analyze samples where heterozygosity is important (e.g., hybrids and allopolyploids).

## Preparing the data

Create a folder for the input, analyses and output. Create a subfolder named `data` that contains the alignments in `.fasta` format, or symbolic links to those alignments. In the code below, change the working directory (wd or WD) to this directory. I will use `~/scratch/test_AmbToHet`.

If you have one or multiple fasta files that need to be combined in a single alignment and transformed into snps, run the code below:
```bash
WD=~/scratch/test_AmbToHet

## load required modules
module load StdEnv/2020
module load snp-sites/2.5.1
conda activate base

cd $WD

## concatenate fasta files in ./data
#seqkit concat --full ./data/*.fas* > concat.aln
catfasta2phyml \
  --concatenate \
  --fasta \
  ./data/*.fas* > concat.aln 2> partitions.txt

## extract snps from the concatenated alignment
snp-sites -m -o snps.aln concat.aln

```

## Importing the SNPs in R and running basic analyses (PCA and NJ)

Run the code below to do a basic analysis in R:
```R
WD=~/scratch/test_AmbToHet

## load required modules
module load StdEnv/2023
module load gcc/12.3 r/4.4.0
export R_LIBS=/project/def-bourret/shared/R/$EBVERSIONR/

cd $WD

R

## load required packages
library("seqinr")
library("adegenet")
library("ade4")
library("ape")
library("factoextra")

## read phylip file into matrix
library("seqinr")
ali <- as.matrix(read.alignment("snps.aln", format="fasta", forceToLower=TRUE))

## convert to SNP table
align_to_alleles <- function(x, ploidy=2) {

    if (is.matrix(x) == FALSE)
        stop("x needs to be a matrix")

    if (ploidy != 2)
        stop("only ploidy=2 is implemented currently")
    
    ## homozygotes
    x[x=="a"] <- "0/0"
    x[x=="c"] <- "1/1"
    x[x=="t"] <- "2/2"
    x[x=="g"] <- "3/3"

    ## heterozygotes
    x[x=="m"] <- "0/1" ## a/c
    x[x=="w"] <- "0/2" ## a/t
    x[x=="r"] <- "0/3" ## a/g
    x[x=="y"] <- "1/2" ## c/t
    x[x=="s"] <- "1/3" ## c/g
    x[x=="k"] <- "2/3" ## t/g

    ## anything else
    x[!grepl("/",x)] <- NA

    return(as.data.frame(x))
}
ali.df <- as.data.frame(align_to_alleles(ali))

## transform dataframe into genind object
library("adegenet")
ali.gen <- df2genind(ali.df, sep="/")

## center/scale allele frequencies and replace zeroes by col mean
ali.scale <- scaleGen(ali.gen, NA.method="mean")

## run a PCA on scaled allele frequencies data
library("ade4")
library("factoextra")
ali.pca <- dudi.pca(ali.scale, scannf=FALSE, nf=3)
jpeg("pca.jpg", width=2000, height=2000)
fviz_pca_ind(ali.pca, repel=TRUE)
dev.off()

## run a neighbor-joining analysis on euclidean distance matrix
library("ape")
ali.d <- dist(ali.gen)
jpeg("nj.jpg", width=2000, height=2000)
plot(nj(ali.d), "unrooted")
dev.off()

q("no")

```

Once this is done, you can download the results (PCA and NJ tree) by running the code below on your local machine:
```bash
LOCAL=~/Downloads
REMOTE=bourret@beluga.computecanada.ca:/home/bourret/scratch/test_AmbToHet

rsync --progress $REMOTE/*.jpg $LOCAL/

```

## Send test data to cluster

Do not run this code, this is only for my own (Étienne L-B's) testing!
```bash
LOCAL=/stor/ELB/UdeM/research/Crataegus/gendist/Assembly/*.fasta
REMOTE=bourret@beluga.computecanada.ca:/home/bourret/scratch/test_AmbToHet/data/

rsync --progress $LOCAL $REMOTE

```

