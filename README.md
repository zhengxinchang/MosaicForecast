# MosaicPC-Calico
A mosaicSNV detecting software based on phasing and random forest

# Versions:
Python version 3.6.1\
R version 3.4.1

# Packages need to be installed:
## Python packages need to import or install:
os\
sys\
collections\
itertools\
subprocess\
multiprocessing\
regex (2017.9.23)\
numpy (1.12.1)\
pyfaidx (0.5.3.1)\
pysam (0.11.2.2)\
pysamstats (1.0.1)\
scipy (0.19.0)\
pandas (0.20.1)
## R packages need to import or install:
caret\
e1071\
glmnet\
nnet\
ggbiplot

# Usage:
## Phasing:
python phase\_WGS.python3.6.1\_multiplesamples.py\
Usage: python phase\_WGS.python3.6.1\_multiplesamples.py bam\_dir output\_dir ref\_fasta n\_jobs input\_positions(file format:chr pos-1 pos ref alt sample, sep=\t) min\_dp\_inforSNPs(integer, can be set to 20)

Note: 1. Name of bam files should be "sample.bam" under the bam\_dir. 2. there should be a fai file under the same dir of the fasta file (samtools faidx input.fa). 3. The "min\_dp\_inforSNPs" is the minimum depth of coverage of trustworthy neaby het SNPs.\
hap=2: het\
hap=3: mosaic\
hap>3: cnv/repeat


Demo:\
python phase\_WGS.python3.6.1\_multiplesamples.py demo demo/phasing ${reference\_dir}/human\_g1k\_v37\_decoy.fasta 1 demo/test.input 20

## Feature extraction:
### 1st step:
python feature\_extraction.python3.6.1.py\
Usage: python(v3.6.1) feature\_extraction.py input\_bed(file\_format: chr pos-1 pos ref alt sample, sep="\t") output\_features bam\_dir reference\_fasta

Note: 1. Name of bam files should be "sample.bam" under the bam\_dir. 2.there should be a fai file under the same dir of the fasta file (samtools faidx input.fa). 3. We did not use dbSNP AF as an feature because we only focus on ultra-rare mosaic mutations, but you can use it to train your model if you have interest in common variants.

Demo:\
python feature\_extraction.python3.6.1.py demo/test.input demo/test.features demo ${reference\_dir}/human\_g1k\_v37\_decoy.fasta

### 2nd step:
Rscript feature\_extraction.R\
Usage: Rscript feature\_extraction.R input\_file(feature\_list\_frompython) output\_file(feature\_list\_R)

Demo:\
Rscript feature\_extraction.R demo/test.features demo/test\_feature\_list\_R

## Prediction:
Rscript prediction.R\
Usage: Rscript prediction.R input\_file(feature\_list) model\_trained output\_file(predictions)

Demo:\
Rscript prediction.R demo/test\_feature\_list\_R models\_trained/250x\_rf\_PCAandPhase\_30mtry.rds demo/test\_predictions

### You may use our models trained with WGS PE data (given different read depths):
50x\_rf\_PCAandPhase\_30mtry.rds\
100x\_rf\_PCAandPhase\_30mtry.rds\
150x\_rf\_PCAandPhase\_30mtry.rds\
200x\_rf\_PCAandPhase\_30mtry.rds\
250x\_rf\_PCAandPhase\_30mtry.rds

## You could also train models using your own data:
### Method1. train models only based on phasable sites:
Rscript train\_on\_phasing.R\
Usage: Rscript train\_on\_phasing.R trainset prediction\_model

Demo:\
Rscript train\_on\_phasing.R demo/all\_putative\_mosaics\_feature.list.features\_addphasing\_addvalidation models\_trained/model\_phasingcorrection.rds

### Method2. correct genotypes from phasing and train models based on corrected phasable sites:
1st step, correct genotype labels of phasable sites using experimentally validated sites:\
Rscript phasing\_correction\_train.R\
Usage: Rscript training\_phasing\_correction.R trainset prediction\_model\_phasingcorrection output\_file\_phasingcorrected

Demo:\
Rscript phasing\_correction\_train.R demo/all\_putative\_mosaics\_feature.list.features\_addphasing\_addvalidation demo/prediction\_model\_phasingcorrection.rds demo/test\_phasingcorrected


2nd step, train based on corrected phasable sites:\

