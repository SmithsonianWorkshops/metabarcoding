## Mothur tutorial

In this tutorial, we will run [Mothur](https://mothur.org/wiki/Main_Page), developed by the Schloss Lab at The University of Michigan. (Schloss, P.D., _et al._, Introducing mothur: Open-source, platform-independent, community-supported software for describing and comparing microbial communities. Appl Environ Microbiol, 2009. 75(23):7537-41.)

Protcol modified from Kozich JJ, Westcott SL, Baxter NT, Highlander SK, Schloss PD. (2013): Development of a dual-index sequencing strategy and curation pipeline for analyzing amplicon sequence data on the MiSeq Illumina sequencing platform. Applied and Environmental Microbiology. 79(17):5112-20. (accessed 9/26/2017).

You can download mothur and run it on your laptop, but we recommend running it on Hydra so we can more easily help you if you run into problems. The only difference when running things on Hydra is you run the commands via job scripts rather than interactively. Once you do this for a few analyses, it will become more comfortable. The informaticcs team is available to answer questions at ```si-hpc@si.edu```

Taxonomic databases are here: 
```/data/genomics/db/metabarcoding/Greengenes_13_8_99.taxonomy```

```/data/genomics/db/metabarcoding/RDP_trainset16_022016.rdp```

```/data/genomics/db/metabarcoding/silva.bacteria```

```/data/genomics/db/metabarcoding/18S```

In the span of this workshop, we will analyze two datasets in mothur, a 16S dataset from the mothur MiSeqSOP tutorial and an 18S dataset from Katrina Lohan. Sequence data are here:
```/pool/genomics/dikowr/mothur_tutorial/16S_data```
```/pool/genomics/dikowr/mothur_tutorial/18S_data```
We recommend copying the entire ```/pool/genomics/dikowr/mothur_tutorial/``` directory to your space (either in ```/pool/genomics``` or ```/pool/biology```). Login to Hydra and use ```cp -r``` to do this.

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
#$ -N mothur_test
#$ -o mothur_test.log
#
# ----------------Modules------------------------- #
module load bioinformatics/mothur/1.39.0
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#
mothur batchfile
#
echo = `date` job $JOB_NAME done
```

The batchfile is where you put your commands, either one at a time or sequentially. You should practice running one command at a time at first, and then try stringing commands together once you complete them successfully alone. Take care to adjust the number of requested CPUs and amount of RAM to reflect the commands.

Here is a list of the commands we will perform and what they do. For further explanation, see (https://mothur.org/wiki/MiSeq_SOP). Note that the following commands are specific to the 16S dataset - commands will need to be adjusted for the 18S dataset.

**QUALITY CONTROL:**
1. make a "stability" file ```make.file```
2. combine pairs of reads for each sample and then combine data from all samples into one file ```make.contigs```
3. summarize the sequences - we will use this command a bunch of times ```summary.seqs```
4. remove sequences that contain ambiguous bases and/or are too long ```screen.seqs```
5. merge duplicates ```unique.seqs```
6. make a table with names of unique sequences and counts per group ```count.seqs```
7. trim reference alignment to only the bases needed and rename this file ```pcr.seqs``` & ```rename.file```
8. align your sequences to the reference ```align.seqs```
9. summarize the sequences again ```summary.seqs```
10. screen sequences for alignment errors ```screen.seqs```
11. remove sequences that are probably non-specific amplification identified with above step ```filter.seqs``` 
12. merge duplicates again ```unique.seqs```
13. merge very similar sequences (_e.g._ allow 1 difference per 100 bp) ```pre.cluster```
14. search for chimeras ```chimera.vsearch```
15. remove identified chimeras ```remove.seqs```

Sample full commands for the above steps:
1. ```make.file(inputdir=16S_data, type=fastq, prefix=stability)```
2. ```make.contigs(file=stability.files, processors=8)```
3. ```summary.seqs(fasta=stability.trim.contigs.fasta)```
4. ```screen.seqs(fasta=stability.trim.contigs.fasta, group=stability.contigs.groups, maxambig=0, maxlength=275)```
5. ```unique.seqs(fasta=stability.trim.contigs.good.fasta)``` 
6. ```count.seqs(name=stability.trim.contigs.good.names, group=stability.contigs.good.groups)``` 
7. ```pcr.seqs(fasta=silva.bacteria.fasta, start=11894, end=25319, keepdots=F, processors=8)``` **copy silva.bacteria.fasta to your space from** ```/data/genomics/db/metabarcoding/silva.bacteria```
8. ```rename.file(input=silva.bacteria.pcr.fasta, new=silva.v4.fasta)```
9. ```align.seqs(fasta=stability.trim.contigs.good.unique.fasta, reference=silva.v4.fasta)```
10. ```summary.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table)``` 
11. ```screen.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table, summary=stability.trim.contigs.good.unique.summary, start=1968, end=11550, maxhomop=8)```
12. ```filter.seqs(fasta=stability.trim.contigs.good.unique.good.align, vertical=T, trump=.)``` 
13. ```unique.seqs(fasta=stability.trim.contigs.good.unique.good.filter.fasta, count=stability.trim.contigs.good.good.count_table)``` 
14. ```pre.cluster(fasta=stability.trim.contigs.good.unique.good.filter.unique.fasta, count=stability.trim.contigs.good.unique.good.filter.count_table, diffs=2)``` 
15. ```chimera.vsearch(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.count_table, dereplicate=t)```
16. ```remove.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.fasta, accnos=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.accnos)```


**TAXONOMIC CLASSIFICATION**
Run these steps in the same batchfile:
1. classify sequences (_e.g._ with RDP training set and Bayesian classifier) ```classify.seqs```
2. remove off-target taxa ```remove.lineage```
3. created updated taxonomy summary ```summary.tax```

Sample full commands for the above steps:
Run these steps in the same batchfile:
```classify.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, reference=/data/genomics/db/metabarcoding/RDP_trainset16_022016.rdp/trainset16_022016.rdp.fasta, taxonomy=/data/genomics/db/metabarcoding/RDP_trainset16_022016.rdp/trainset16_022016.rdp.tax, cutoff=80)```
```remove.lineage(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.rdp.wang.taxonomy, taxon=Chloroplast-Mitochondria-unknown-Archaea-Eukaryota)```
```summary.tax(taxonomy=current, count=current)```

**CLUSTERING**
1. calaulate pairwise distances between sequences with your cutoff ```dist.seqs```
2. cluster sequences into OTUs ```cluster```
3. determine how many sequences are in each OTU according to your cutoff ```make.shared```
4. classify OTUs```classify.otu```
5. build a tree ```clearcut```
6. rename files to simplify ```rename.file```
7. count how many sequences are in each sample ```count.groups```
8. generate a subsampled file for our analyses ```sub.sample```

Sample full commands for the above steps:
1. ```dist.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.fasta, cutoff=0.03)```
2. ```cluster(column=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.dist, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table)```
3. ```make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, label=0.03)```
4. ```classify.otu(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.rdp.wang.pick.taxonomy, label=0.03)```
5. ```dist.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.fasta, output=lt, processors=8)```
6. ```clearcut(phylip=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.phylip.dist)```
7. ```rename.file(count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, tree=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.phylip.tre, shared=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.shared, constaxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.0.03.cons.taxonomy)```
8. ```count.groups(shared=stability.opti_mcc.shared)sub.sample(shared=stability.opti_mcc.shared, size=2392)```

**ALPHA DIVERSITY:**
1. generate rarefaction curves ```rarefaction.single```
2. make a table containing the number of sequences, the sample coverage, observed OTUs, and Inverse Simpson diversity estimate ```summary.single```
3. calculate the total of the unique branch length in the tree ```phylo.diversity```

Sample full commands for the above steps:
1. ```rarefaction.single(shared=stability.opti_mcc.shared, calc=sobs, freq=100)```
2. ```summary.single(shared=stability.opti_mcc.shared, calc=nseqs-coverage-sobs-invsimpson, subsample=2392)```
3. ```phylo.diversity(tree=stability.tre, count=stability.count_table, rarefy=T)```

**BETA DIVERSITY:**
1. generate a heatmap to look at relative abundance of OTUs across samples ```heatmap.bin```
2. calculate similarity of membership and structure of samples ```dist.shared```
3. generate a heatmap to visualize the above ```heatmap.sim```
4. generate a venn diagram ```venn```

Sample full commands for the above steps:
1. ```heatmap.bin(shared=stability.opti_mcc.0.03.subsample.shared, scale=log2, numotu=50)```  
2. ```dist.shared(shared=stability.opti_mcc.shared, calc=thetayc-jclass, subsample=2392)```
3. ```mothur > heatmap.sim(phylip=stability.opti_mcc.jclass.0.03.lt.ave.dist)```
4. ```venn(shared=stability.opti_mcc.0.03.subsample.shared, groups=F3D0-F3D1-F3D2-F3D3)```