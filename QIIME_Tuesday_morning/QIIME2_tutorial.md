
## QIIME 2 Tutorial


In this tutorial, we will run part of the [QIIME 2](https://docs.qiime2.org/2017.9/tutorials/moving-pictures/) Moving Pictures tutorial on Hydra. A study based on these samples was originally published in Caporaso et al. (2011). The data used in this tutorial were sequenced on an Illumina HiSeq using the Earth Microbiome Project hypervariable region 4 (V4) 16S rRNA sequencing protocol.


*Download data to Hydra*
1. change directory to your space on Hydra (e.g. ```cd /pool/genomics/USER```)
2. make a new directory for the data (e.g. ```mkdir emp-single-end-sequences```) and ```cd``` into that new directory.
3. use ```wget``` to download barcodes (```wget https://data.qiime2.org/2017.9/tutorials/moving-pictures/emp-single-end-sequences/barcodes.fastq.gz```)
4. use ```wget``` to download sequences (```wget https://data.qiime2.org/2017.9/tutorials/moving-pictures/emp-single-end-sequences/sequences.fastq.gz```)
5. ```cd ..```
6. use ```wget``` to download sample metadata (```wget https://data.qiime2.org/2017.9/tutorials/moving-pictures/sample_metadata.tsv```)


**Import data into QIIME**
All data that is used as input to QIIME 2 is in form of QIIME 2 artifacts, which contain information about the type of data and the source of the data. So, the first thing we need to do is import these sequence data files into a QIIME 2 artifact.

**SAMPLE JOB FILE:**
```
# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q sThC.q
#$ -pe mthread 2
#$ -l mres=4G,h_data=4G,h_vmem=4G
#$ -cwd
#$ -j y
#$ -N qiime_test
#$ -o qiime_test.log
#
# ----------------Modules------------------------- #
module load bioinformatics/qiime2/2017.8
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#
qiime tools import \
  --type EMPSingleEndSequences \
  --input-path emp-single-end-sequences \
  --output-path emp-single-end-sequences.qza
#
echo = `date` job $JOB_NAME done
```
To view .qza and .qzv files, try this: https://view.qiime2.org/

**Demultiplex sequences**
```
qiime demux emp-single \
  --i-seqs emp-single-end-sequences.qza \
  --m-barcodes-file sample_metadata.tsv \
  --m-barcodes-category BarcodeSequence \
  --o-per-sample-sequences demux.qza
 ```

**Summarize demultiplexed sequences**
```
qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv
  ```
View demux.qzv here: https://view.qiime2.org/ and look to see at which base you should trim.

**DADA2 quality control**
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left 0 \
  --p-trunc-len 120 \
  --o-representative-sequences rep-seqs-dada2.qza \
  --o-table table-dada2.qza
  ```
**Generate FeatureTable and FeatureData Summaries**
The feature-table summarize command will give you information on how many sequences are associated with each sample and with each feature, histograms of those distributions, and some related summary statistics. The feature-table tabulate-seqs command will provide a mapping of feature IDs to sequences, and provide links to easily BLAST each sequence against the NCBI nt database. 

```
qiime feature-table summarize \
  --i-table table-dada2.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample_metadata.tsv
qiime feature-table tabulate-seqs \
  --i-data rep-seqs-dada2.qza \
  --o-visualization rep-seqs.qzv
 ```

**Generate a tree for phylogenetic diversity analyses**
1. alignment with mafft
 ```
 qiime alignment mafft \
  --i-sequences rep-seqs-dada2.qza \
  --o-alignment aligned-rep-seqs.qza
  ```

 2. mask alignment for highly variable positions
 ```
 qiime alignment mask \
  --i-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza
  ```

3. generate a tree with fasttree
```
qiime phylogeny fasttree \
  --i-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza
  ```

4. apply a midpoint root to your tree
```
qiime phylogeny midpoint-root \
  --i-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
  ```

**Taxonomic analysis**

The GreenGenes classifier is here: ```/data/genomics/db/metabarcoding/qiime_gg_classifier/gg-13-8-99-515-806-nb-classifier.qza```. This classifier was trained on the Greengenes 13_8 99% OTUs, where the sequences have been trimmed to only include 250 bases from the region of the 16S that was sequenced in this analysis (the V4 region, bound by the 515F/806R primer pair). We’ll apply this classifier to our sequences, and we can generate a visualization of the resulting mapping from sequence to taxonomy.

1. classify sequences
```
qiime feature-classifier classify-sklearn \
  --i-classifier /data/genomics/db/metabarcoding/qiime_gg_classifier/gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza
 ```
2. tabulate metadata
```
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
 ```
3. create a barplot that displays your classification
```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample_metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
 ```

**Alpha diversity**
1. Use ```core-metrics-phylogenetic```, which rarefies a FeatureTable to a user-specified depth, and then computes a series of alpha and beta diversity metrics. 
```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 1109 \
  --output-dir core-metrics-results
 ```
 There are lots of outputs here: look at them with ```ls``` and with the QIIME visualizer. Also see the QIIME tutorial webpage for further details.

