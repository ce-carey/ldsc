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
___

# LDSC (LD SCore) `v1.0.0`

`ldsc` is a command line tool for estimating heritability and genetic correlation from GWAS summary statistics. `ldsc` also computes LD Scores.

## Getting Started



In order to download `ldsc`, you should clone this repository via the commands
```  
git clone https://github.com/bulik/ldsc.git
cd ldsc
```

In order to install the Python dependencies, you will need the [Anaconda](https://store.continuum.io/cshop/anaconda/) Python distribution and package manager. After installing Anaconda, run the following commands to create an environment with LDSC's dependencies:

```
conda env create --file environment.yml
source activate ldsc
```

Once the above has completed, you can run:

```
./ldsc.py -h
./munge_sumstats.py -h
```
to print a list of all command-line options. If these commands fail with an error, then something as gone wrong during the installation process. 

Short tutorials describing the four basic functions of `ldsc` (estimating LD Scores, h2 and partitioned h2, genetic correlation, the LD Score regression intercept) can be found in the wiki. If you would like to run the tests, please see the wiki.

## Updating LDSC

You can update to the newest version of `ldsc` using `git`. First, navigate to your `ldsc/` directory (e.g., `cd ldsc`), then run
```
git pull
```
If `ldsc` is up to date, you will see 
```
Already up-to-date.
```
otherwise, you will see `git` output similar to 
```
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/bulik/ldsc
   95f4db3..a6a6b18  master     -> origin/master
Updating 95f4db3..a6a6b18
Fast-forward
 README.md | 15 +++++++++++++++
 1 file changed, 15 insertions(+)
 ```
which tells you which files were changed. If you have modified the `ldsc` source code, `git pull` may fail with an error such as `error: Your local changes to the following files would be overwritten by merge:`. 

In case the Python dependencies have changed, you can update the LDSC environment with

```
conda env update --file environment.yml
```

## Where Can I Get LD Scores?

You can download [European](https://data.broadinstitute.org/alkesgroup/LDSCORE/eur_w_ld_chr.tar.bz2) and [East Asian LD Scores](https://data.broadinstitute.org/alkesgroup/LDSCORE/eas_ldscores.tar.bz2) from 1000 Genomes [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/). These LD Scores are suitable for basic LD Score analyses (the LD Score regression intercept, heritability, genetic correlation, cross-sex genetic correlation). You can download partitioned LD Scores for partitioned heritability estimation [here](http://data.broadinstitute.org/alkesgroup/LDSCORE/).


## Support

Before contacting us, please try the following:

1. The [wiki](https://github.com/bulik/ldsc/wiki) has tutorials on [estimating LD Score](https://github.com/bulik/ldsc/wiki/LD-Score-Estimation-Tutorial), [heritability, genetic correlation and the LD Score regression intercept](https://github.com/bulik/ldsc/wiki/Heritability-and-Genetic-Correlation) and [partitioned heritability](https://github.com/bulik/ldsc/wiki/Partitioned-Heritability).
2. Common issues are described in the [FAQ](https://github.com/bulik/ldsc/wiki/FAQ)
2. The methods are described in the papers (citations below)

If that doesn't work, you can get in touch with us via the [google group](https://groups.google.com/forum/?hl=en#!forum/ldsc_users).

Issues with LD Hub?  Email ld-hub@bristol.ac.uk


## Citation

If you use the software or the LD Score regression intercept, please cite

[Bulik-Sullivan, et al. LD Score Regression Distinguishes Confounding from Polygenicity in Genome-Wide Association Studies.
Nature Genetics, 2015.](http://www.nature.com/ng/journal/vaop/ncurrent/full/ng.3211.html)

For genetic correlation, please also cite

Bulik-Sullivan, et al. An Atlas of Genetic Correlations across Human Diseases and Traits. bioRxiv doi: http://dx.doi.org/10.1101/014498

For partitioned heritability, please also cite

Finucane, HK, et al. Partitioning Heritability by Functional Category using GWAS Summary Statistics. bioRxiv doi: http://dx.doi.org/10.1101/014241

If you find the fact that LD Score regression approximates HE regression to be conceptually useful, please cite

Bulik-Sullivan, Brendan. Relationship between LD Score and Haseman-Elston, bioRxiv doi http://dx.doi.org/10.1101/018283

For LD Hub, please cite

Zheng, et al. LD Hub: a centralized database and web interface to perform LD score regression that maximizes the potential of summary level GWAS data for SNP heritability and genetic correlation analysis. Bioinformatics (2016)
https://doi.org/10.1093/bioinformatics/btw613


## License

This project is licensed under GNU GPL v3.


## Authors

Brendan Bulik-Sullivan (Broad Institute of MIT and Harvard)

Hilary Finucane (MIT Department of Mathematics)
