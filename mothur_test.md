###Mothur tutorial

In this tutorial, we will run Mothur [mothur_site](https://mothur.org/wiki/Main_Page), developed by the Schloss Lab at The University of Michigan. (Schloss, P.D., _et al._, Introducing mothur: Open-source, platform-independent, community-supported software for describing and comparing microbial communities. Appl Environ Microbiol, 2009. 75(23):7537-41.)

Protcol modified from Kozich JJ, Westcott SL, Baxter NT, Highlander SK, Schloss PD. (2013): Development of a dual-index sequencing strategy and curation pipeline for analyzing amplicon sequence data on the MiSeq Illumina sequencing platform. Applied and Environmental Microbiology. 79(17):5112-20. (accessed 9/26/2017).

You can download mothur and run it on your laptop, but we recommend running it on Hydra so we can more easily help you if you run into problems. The only difference when running things on Hydra is you run the commands via job scripts rather than interactively. Once you do this for a few analyses, it will become more comfortable. The informaticcs team is available to answer questions at ```si-hpc@si.edu```

Taxonomic databases are here: 
```/data/genomics/db/metabarcoding/Greengenes_13_8_99.taxonomy```
```/data/genomics/db/metabarcoding/RDP_trainset16_022016.rdp```
```/data/genomics/db/metabarcoding/silva.bacteria```

Sequence data are here:
```/pool/genomics/dikowr/mothur_tutorial```
LIST FILES

Here is a list of the commands we will perform and what they do. For further explanation, see (https://mothur.org/wiki/MiSeq_SOP):

QUALITY CONTROL:
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

make.file(inputdir=MiSeq_SOP, type=fastq, prefix=stability)
make.contigs(file=stability.files, processors=8)
summary.seqs(fasta=stability.trim.contigs.fasta) 
screen.seqs(fasta=stability.trim.contigs.fasta, group=stability.contigs.groups, maxambig=0, maxlength=275) 
unique.seqs(fasta=stability.trim.contigs.good.fasta) 
count.seqs(name=stability.trim.contigs.good.names, group=stability.contigs.good.groups) 
pcr.seqs(fasta=silva.bacteria.fasta, start=11894, end=25319, keepdots=F, processors=8) 
rename.file(input=silva.bacteria.pcr.fasta, new=silva.v4.fasta)
align.seqs(fasta=stability.trim.contigs.good.unique.fasta, reference=silva.v4.fasta)
summary.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table) 
screen.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table, summary=stability.trim.contigs.good.unique.summary, start=1968, end=11550, maxhomop=8)
filter.seqs(fasta=stability.trim.contigs.good.unique.good.align, vertical=T, trump=.) 