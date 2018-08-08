# Bioinformatics Reconciliation for LDpred

These scripts were modified from the original LDSC repo for the purpose of bioinformatics reconciliation of summary statistics for use with [LDpred](https://github.com/bvilhjal/ldpred). [munge_sumstats.py](https://github.com/bulik/ldsc/blob/master/munge_sumstats.py) is a fantastic little script, and rather than reinventing the wheel and creating my own reconciliation script, I decided to heavily modify it to output the BASIC input format for [LDpred](https://github.com/bvilhjal/ldpred). A full list of line-by-line modifications can be found in [diff.log](https://github.com/ce-carey/process-sumstats/blob/master/diff.log). 

NOTE: This is under active development and generally intended for my own use and the use of my collaborators. USE AT YOUR OWN PERIL!! 

## Setup

As with the original [LDSC](https://github.com/bulik/ldsc), process-sumstats can be installed and run as follows:

1. Clone this repo:
```
git clone https://github.com/ce-carey/process-sumstats.git
cd process-sumstats
```
2. Install [Anaconda](https://www.anaconda.com/download/) and create an environment with the required dependencies:
```
conda env create --file environment.yml
source activate process-sumstats
```
3. Check that things are installed correctly by listing the commandline options:
```
./process_sumstats.py -h
```

## Input

The original LDSC wiki has a great [tutorial](https://github.com/bulik/ldsc/wiki/Heritability-and-Genetic-Correlation) that describes how to use the original script ([munge_sumstats.py](https://github.com/bulik/ldsc/blob/master/munge_sumstats.py)). However, my script requires additional inputs in order to be compatible with LDpred:

1. A unique identifier (e.g., the rs number)
2. The chromosome number of the variant
3. The basepair position of the variant
2. Allele 1 (effect allele)
3. Allele 2 (non-effect allele)
4. Sample size (which often varies from SNP to SNP)
5. A P-value
6. A effect allele weight (beta, OR, log odds, Z-score, etc)

The `process_sumstats.py` script should recognize most standard headers for these required columns, which do _not_ need to be in the order listed above. 

## Output

Output of this script is a file, OUTPUTNAME.basic, in a slightly-adapted LDpred BASIC format, an example of which is adapted from the [LDpred README](https://github.com/bvilhjal/ldpred) below:

CHR | SNP | A1 | A2 | BP | WEIGHT | P       
--- | --- | --- | --- | --- | --- | ---
chr1 | rs4951859 | C | G | 729679 | -0.02170 | 0.2083  
chr1 | rs142557973 | T | C | 731718 | 0.01930 | 0.3298  

The main difference between the above-described format and the original LDpred BASIC format is that I've included an effect allele weight rather than an OR. All OR's in the original sumstats files will be automatically converted to log(OR). I have altered this input, and correspondingly [my fork of the LDpred repo](https://github.com/ce-carey/ldpred), to use a weight rather than an OR in order to allow for a greater variety of sumstat inputs (e.g., both OR's from case/control studies and betas from studies of continuous traits). 

## Options

A full list of options can be accessed by typing `./process_sumstats.py -h`. These are reprinted below for convenience:

```
usage: process_sumstats.py [-h] [--sumstats SUMSTATS] [--N N] [--N-cas N_CAS]
                           [--N-con N_CON] [--out OUT] [--info-min INFO_MIN]
                           [--maf-min MAF_MIN] [--daner] [--daner-n]
                           [--no-alleles] [--merge-alleles MERGE_ALLELES]
                           [--n-min N_MIN] [--chunksize CHUNKSIZE] [--snp SNP]
                           [--chr CHR] [--bp BP] [--N-col N_COL]
                           [--N-cas-col N_CAS_COL] [--N-con-col N_CON_COL]
                           [--a1 A1] [--a2 A2] [--p P] [--frq FRQ]
                           [--weights WEIGHTS] [--info INFO]
                           [--info-list INFO_LIST] [--nstudy NSTUDY]
                           [--nstudy-min NSTUDY_MIN] [--ignore IGNORE]
                           [--a1-inc] [--keep-maf]

optional arguments:
  -h, --help            show this help message and exit
  --sumstats SUMSTATS   Input filename.
  --N N                 Sample size If this option is not set, will try to
                        infer the sample size from the input file. If the
                        input file contains a sample size column, and this
                        flag is set, the argument to this flag has priority.
  --N-cas N_CAS         Number of cases. If this option is not set, will try
                        to infer the number of cases from the input file. If
                        the input file contains a number of cases column, and
                        this flag is set, the argument to this flag has
                        priority.
  --N-con N_CON         Number of controls. If this option is not set, will
                        try to infer the number of controls from the input
                        file. If the input file contains a number of controls
                        column, and this flag is set, the argument to this
                        flag has priority.
  --out OUT             Output filename prefix.
  --info-min INFO_MIN   Minimum INFO score.
  --maf-min MAF_MIN     Minimum MAF.
  --daner               Use this flag to parse Stephan Ripke's daner* file
                        format.
  --daner-n             Use this flag to parse more recent daner* formatted
                        files, which include sample size column 'Nca' and
                        'Nco'.
  --no-alleles          Don't require alleles. Useful if only unsigned summary
                        statistics are available and the goal is h2 /
                        partitioned h2 estimation rather than rg estimation.
  --merge-alleles MERGE_ALLELES
                        Same as --merge, except the file should have three
                        columns: SNP, A1, A2, and all alleles will be matched
                        to the --merge-alleles file alleles.
  --n-min N_MIN         Minimum N (sample size). Default is (90th percentile
                        N) / 2.
  --chunksize CHUNKSIZE
                        Chunksize.
  --snp SNP             Name of SNP column (if not a name that ldsc
                        understands). NB: case insensitive.
  --chr CHR             Name of CHR column (if not a name that ldsc
                        understands). NB: case insensitive.
  --bp BP               Name of BP column (if not a name that ldsc
                        understands). NB: case insensitive.
  --N-col N_COL         Name of N column (if not a name that ldsc
                        understands). NB: case insensitive.
  --N-cas-col N_CAS_COL
                        Name of N column (if not a name that ldsc
                        understands). NB: case insensitive.
  --N-con-col N_CON_COL
                        Name of N column (if not a name that ldsc
                        understands). NB: case insensitive.
  --a1 A1               Name of A1 column (if not a name that ldsc
                        understands). NB: case insensitive.
  --a2 A2               Name of A2 column (if not a name that ldsc
                        understands). NB: case insensitive.
  --p P                 Name of p-value column (if not a name that ldsc
                        understands). NB: case insensitive.
  --frq FRQ             Name of FRQ or MAF column (if not a name that ldsc
                        understands). NB: case insensitive.
  --weights WEIGHTS     Name of weight column, comma null value (e.g., Z,0 or
                        OR,1). NB: case insensitive.
  --info INFO           Name of INFO column (if not a name that ldsc
                        understands). NB: case insensitive.
  --info-list INFO_LIST
                        Comma-separated list of INFO columns. Will filter on
                        the mean. NB: case insensitive.
  --nstudy NSTUDY       Name of NSTUDY column (if not a name that ldsc
                        understands). NB: case insensitive.
  --nstudy-min NSTUDY_MIN
                        Minimum # of studies. Default is to remove everything
                        below the max, unless there is an N column, in which
                        case do nothing.
  --ignore IGNORE       Comma-separated list of column names to ignore.
  --a1-inc              A1 is the increasing allele.
  --keep-maf            Keep the MAF column (if one exists).
```
