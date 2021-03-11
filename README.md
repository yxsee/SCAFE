
 ```shell
                        O~~~AA      O~~         O~       O~~~~~~~AO~~~~~~~~A
                      O~~    O~~ O~~   O~~     O~O~~     O~~      O~~       
                       O~~      O~~           O~  O~~    O~~      O~~       
                         O~~    O~~          O~~   O~~   O~~~~~AA O~~~~~~A  
                            O~~ O~~         O~~~~~A O~~  O~~      O~~       
                      O~~    O~~ O~~   O~~ O~~       O~~ O~~      O~~       
                        O~~~~A     O~~~   O~~         O~~O~~      O~~~~~~~AA
 ```

**SCAFE** (Single Cell Analysis of Five' End) is a tool suite for processing of single cell 5’end RNA-seq data. It takes a read alignment file (*.bam) from single-cell 5'end RNA sequencing, precisely identifies the read 5'ends and removes strand invasion artefacts, performs TSS clustering and filters for genuine TSS clusters using logistic regression, defines transcribed cis-regulatory elements (tCRE) and annotated them to gene models. It counts the UMI in tCRE in single cells and returns a tCRE UMI/cellbarcode matrix ready for downstream analyses. 

## Citing SCAFE

Profiling of transcribed cis-regulatory elements in single cells. _bioRxiv_, 2021, [XXXXXX](https://XXXXXXXXXX/)

## What does SCAFE do?
### SCAFE detects tCRE activities in single-cells
<div style="text-align:center"><img src=".github/images/tCRE_definition.png?" width="860"></div>
<div style="text-align:center"><img src=".github/images/zenbu.png?" width="860"></div>

Profiling of cis-regulatory elements (CREs, mostly promoters and enhancers) in single cells allows us to interrogate the cell-type specific contexts of gene regulation and genetic predisposition to diseases. Single-cell RNA-5’end-sequencing (sc-end5-seq) methods can detect transcribed CREs (tCREs), enabling the quantification of promoter and enhancer activities in single cells. To identify genuine tCREs, we implemented a workflow which effectively eliminates false positives from sc-end5-seq data.

Blablabla

## How does SCAFE do it?
### [SCAFE Core Tools and Workflows](scripts/)
<div style="text-align:center"><img src=".github/images/flowchart.png?" width="860"></div>

SCAFE consists of [a set of perl programs](scripts/) for processing of single cell 5’end RNA-seq data. Major tools are listed here. SCAFE accepts read alignment in .bam format from standard 10X GenomicsTM tool cellranger. Tool bam_to_ctss extracts the 5’ position of reads, taking the 5’ unencoded-Gs into account. Tool remove_strand_invader removes read 5’ends that are strand invasion artifacts by aligning the TS oligo sequence to the immediate upstream sequence of the read 5’end. Tool cluster performs clustering of read 5’ends using 3rd-party tool Paraclu. Tool filter extracts the properties of TSS clusters and performs multiple logistic regression to distinguish genuine TSS clusters from artifacts. Tool annotate define tCREs by merging closely located TSS clusters and annotate tCREs based on their proximity to known genes. Tool count counts the number of UMI within each tCRE in single cells and generates a tCRE-Cell UMI count matrix. SCAFE tools were also implemented workflows for processing of individual samples or pooling of multiple samples.

Blablabla

## Dependencies

### perl
SCAFE is mainly written in perl (v5.24.1 or later). All scripts are standalone applications and **DO NOT require installations** of extra perl modules. Check whether perl is properly installed on your system.

```shell
#--- Check your perl version
perl --version
```

### R
SCAFE relies on R for logistic regression, ROC analysis and graph plotting. Rscript (v3.6.1 or later) and the following R packages have to be properly installed:

* [ROCR](https://cran.r-project.org/web/packages/ROCR/readme/README.html)
* [PRROC](https://cran.r-project.org/web/packages/PRROC/index.html)
* [caret](https://cran.r-project.org/web/packages/caret/index.html)
* [e1071](https://cran.r-project.org/web/packages/e1071/index.html)
* [ggplot2](https://ggplot2.tidyverse.org/)
* [scales](https://cran.r-project.org/web/packages/scales/index.html)
* [reshape2](https://cran.r-project.org/web/packages/reshape2/index.html)

Failed to install these packages will crash ./scripts/tool.cm.filter.

```shell
#--- Check your Rscript version
Rscript --version
```
### Other 3rd party applications
SCAFE also relies on a number of 3rd party applications. The binaries and executables (Linux) of these applications are distributed with this reprository in the directory ./resources/bin and **DO NOT require installations**.

* [bigWigAverageOverBed](https://github.com/ENCODE-DCC/kentUtils)
* [bedGraphToBigWig](https://github.com/ENCODE-DCC/kentUtils)
* [bedtools](https://bedtools.readthedocs.io/en/latest/)
* [samtools](http://www.htslib.org/)
* [paraclu](http://cbrc3.cbrc.jp/~martin/paraclu/)
* [paraclu-cut.sh](http://cbrc3.cbrc.jp/~martin/paraclu/)

### OS
SCAFE was developed and tested on Debian GNU/Linux 9. Running SACFE on other OS are not guranteed.

## Getting started
Once you ensured the above required R packages are installed, you are all set. Here's the instructions on getting SCAFE, downloading demo data and reference genome, and finally a couple of test runs on the demo data.

### Clone this respository

```shell
#--- make a directory to install SCAFE
mkdir -pm 755 /my/path/to/install/
cd /my/path/to/install/

#--- Obtain SCAFE from github
git clone https://github.com/chung-lab/SCAFE
cd SCAFE

#--- making sure the scripts and binaries are executable
chmod 755 -R ./scripts/
chmod 755 -R ./resources/bin/
```
### Download demo data and reference genome

```shell
#--- download the demo data using script download.demo.input
./scripts/download.demo.input
	
#--- download the reference genome hg19.gencode_v32lift37 for testing demo data
./scripts/download.resources.genome --genome=hg19.gencode_v32lift37
```

### Test run a single cell solo workflow for demo data
```shell
#--- check out the help message of workflow.sc.solo
./scripts/workflow.sc.solo --help

#--- run the workflow on the demo.cellranger.bam, it'll take a couple of minutes 
./scripts/workflow.sc.solo \
--overwrite=yes \
--run_bam_path=./demo/input/sc.solo/demo.cellranger.bam \
--run_cellbarcode_path=./demo/input/sc.solo/demo.barcodes.tsv.gz \
--genome=hg19.gencode_v32lift37 \
--run_tag=demo \
--run_outDir=./demo/output/sc.solo/

#--- check the output of tCRE annotation tables
ls -alh ./demo/output/sc.solo/annotate/demo/log

#--- check the output of tCRE UMI count matrix
ls -alh ./demo/output/sc.solo/count/demo/matrix
```

### Test run for making a custom reference genome
```shell
#--- check out the help message of tool.cm.prep_genome
./scripts/tool.cm.prep_genome --help

#--- run the tool on the TAIR10 assembly with gene model AtRTDv2 
./scripts/tool.cm.prep_genome \
--overwrite=yes \
--gtf_path=./demo/input/genome/TAIR10.AtRTDv2.gtf.gz \
--fasta_path=./demo/input/genome/TAIR10.genome.fa.gz \
--chrom_list_path=./demo/input/genome/TAIR10.chrom_list.txt \
--mask_bed_path=./demo/input/genome/TAIR10.ATAC.bed.gz \
--outputPrefix=TAIR10.AtRTDv2 \
--outDir=./demo/output/genome/
```
### Test run all workflows on bulk and single cell data
```shell
#--- check out the help message of demo.test.run
./scripts/demo.test.run --help

#--- run the all six available workflows on the demo bulk and single cell, it'll take a around 20 minutes
./scripts/demo.test.run \
--overwrite=yes \
--run_outDir=./demo/output/
```

## Run SCAFE for your own data
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum

```shell
run demo

```
