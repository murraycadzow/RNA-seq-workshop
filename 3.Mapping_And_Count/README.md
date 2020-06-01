# How to align sequencing reads to genome

## Preparation of the genome

To be able to map (align) sequencing reads on the genome, the genome needs to be indexed first. In this workshop we will use [HISAT2](https://www.nature.com/articles/nmeth.3317).
Note for speed reason, the reads will be aligned on the chr5 of the mouse genome.

```
cd /mnt/RNAseq_Workshop_Data/genome
#to list what is in your directory:
ls /mnt/RNAseq_Workshop_Data/genome/
chr5.fa
Ensembl_Mus_musculus.GRCm38.94_ch5.gtf

# only chromosome 5 fasta and gtf available in this directory

# index file:
hisat2-build -p 2 -f /mnt/RNAseq_Workshop_Data/genome/chr5.fa /mnt/RNAseq_Workshop_Data/genome/chr5

# take around 3 mins
#list what is in the directory:
ls /mnt/RNAseq_Workshop_Data/genome/
chr5.1.ht2  chr5.3.ht2  chr5.5.ht2  chr5.7.ht2  chr5.fa
chr5.2.ht2  chr5.4.ht2  chr5.6.ht2  chr5.8.ht2  Ensembl_Mus_musculus.GRCm38.94_ch5.gtf

```
Option info:
  * -p number of threads
  * -f fasta file

How many files were created during the indexing process?

## Alignment on the genome (chromosome5)

Now that the genome is prepared. Sequencing reads can be aligned.

Information required:

  * Where the sequence information is stored (e.g. fastq files ...) ?
  * What kind of sequencing: Single End or Paired end ?
  * Where are stored the indexes and the genome? 
  * Where will the mapping files be stored?

```
# syntax:
hisat2 -x /mnt/RNAseq_Workshop_Data/genome/chr5 -1 trimmed/WT1_R1_trimmed.fastq -2 trimmed/WT1_R2_trimmed.fastq  -S mapping/WT1.sam

```
Now we need to align all the rest of the samples.

Hint: looping over the files?


```
# let's use a counter to process our samples:
# for each iteration $i will change:  first iteration i=1, then 2 then 3.
for i in `seq 1 3`; 
do
# processing WT first:
hisat2 -x /mnt/RNAseq_Workshop_Data/genome/chr5 -1 trimmed/WT$i\_R1_trimmed.fastq -2 trimmed/WT$i\_R2_trimmed.fastq  -S mapping/WT$i.sam
# processing KO
hisat2 -x /mnt/RNAseq_Workshop_Data/genome/chr5 -1 trimmed/KO$i\_R1_trimmed.fastq -2 trimmed/KO$i\_R2_trimmed.fastq  -S mapping/KO$i.sam

done

```

Now we can explore our SAM files.

## Converting SAM files to BAM files

This SAM to BAM files can be achieved by several tools. Today we will focus on samtools which the most commonly used.

To convert sam to bam 

```
samtools view -b mapping/WT1.sam -o mapping/WT1.bam

```

We need to perform this step for all the samples. 

Hint: loop???

```
# let's re-use a counter to process our samples:
# for each iteration $i will change:  first iteration i=1, then 2 then 3.
for i in `seq 1 3`;
do
# processing WT first:
samtools view -b mapping/WT$i.sam -o  mapping/WT$i.bam
# processing KO
samtools view -b mapping/KO$i.sam -o  mapping/KO$i.bam
done

```
## Some stats on your mapping:

```

samtools flagstat mapping/WT1.bam
262944 + 0 in total (QC-passed reads + QC-failed reads)
35218 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
260442 + 0 mapped (99.05% : N/A)
227726 + 0 paired in sequencing
113863 + 0 read1
113863 + 0 read2
220394 + 0 properly paired (96.78% : N/A)
222892 + 0 with itself and mate mapped
2332 + 0 singletons (1.02% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)

```
More stats ...

```

samtools stat mapping/WT1.bam

```

## Vizualisation using IGV:

Prior to see the alignment, the mapping file (e.i. BAM) should be sorted and indexed to gain efficiency. This can be done using samtools.

```
for i in `seq 1 3`;
do
# processing WT first:
samtools sort -o mapping/WT$i\_sorted.bam mapping/WT$i.bam
samtools index mapping/WT$i\_sorted.bam
# processing KO
samtools sort -o mapping/KO$i\_sorted.bam mapping/KO$i.bam
samtools index  mapping/KO$i\_sorted.bam
done

```
Now you can open IGV and load the bam files.
to do so in terminal:

```
igv 
```
Then you should load the appropriate genome: mm10. 

Note:
According to the paper, KO construction is *"A loss of function mutation of Gtf2ird1 was generated by a random insertion of a Myc transgene into the region, resulting in a 40 kb deletion surrounding exon 1"*

load your sorted bam file: for WT1 and KO1 click on File then Load from File ... and browse to your bam files.

Now have a look at the gene Gtf2ird1 and zoom in slightly to see the alignment: chr5:134,387,384-134,426,993

  * What do you see?
  * What strand is the library type?

if you don't know what is the library type then you can guess according to the mapping. Here is how you can do it:

  * right click on the WT1_sorted.bam alignment
  * click on Group alignment by then first in pair strand
  * click on Color alignment by then first in pair strand

What do you see?

# Counting
- We need to do some counting!
- Want to generate count data for each gene (actually each exon) - how many reads mapped to each exon in the genome, from each of our samples?
- Once we have that information, we can start thinking about how to determine which genes were differentially expressed in our study.

## Subread and FeatureCounts
- The featureCounts tool from the Subread package can be used to count how many reads aligned to each genome feature (exon).
- Need to specify the annotation informatyion (.gtf file) 
You can process all the samples at once:

```
featureCounts -a Genome/Saccharomyces_cerevisiae.R64-1-1.99.gtf \ -o yeast-chr1_counts.txt *.sam
```

Or you can do process each sample individually:

```
featureCounts -a Genome/Saccharomyces_cerevisiae.R64-1-1.99.gtf -o SRR014335-chr1_counts.txt SRR014335-chr1-Aligned.out.sam
```

 ·
 
