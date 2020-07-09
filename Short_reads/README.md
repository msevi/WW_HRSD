Short read processing
================

Renaming ???
------------

Need metadata info

FastP (pre processing)
----------------------

A detailed description of the software can be found on the developer's <a href="https://github.com/OpenGene/fastp" target="_blank">github</a> Note: Qscores of <a href="https://www.illumina.com/content/dam/illumina-marketing/documents/products/appnotes/novaseq-hiseq-q30-app-note-770-2017-010.pdf" target="_blank">NovaSeq</a>

``` bash
####  Create new environment #### 
conda create -n fastp_env

####  Activate environment and install fastp #### 
conda activate fastp_env
conda install -c bioconda fastp

#### Script #### 
#!/bin/sh
#SBATCH --array=1-1
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=20
#SBATCH --time=24:00:00
#SBATCH --mem=250000
#SBATCH --job-name=fastp
#SBATCH --error=job.%a_%x_%j.stderr
#SBATCH --output=job.%a_%x_%j.stdout
# --dependency=singleton
#SBATCH --partition=short

source ~/miniconda3/bin/activate
conda activate fastp_env
mkdir r_F02 
cd r_F02
fastp --thread 20 \
--in1 /scratch/m.sevi/processing/WW_HRSD/data/short/novaseq_run2/r_F02_1.fq.gz \
--out1 r_F02_1.paired.R1.fq \
--in2 /scratch/m.sevi/processing/WW_HRSD/data/short/novaseq_run2/r_F02_2.fq.gz \
--out2 r_F02_2.paired.R2.fq \
--trim_poly_g \
--trim_poly_x \
--qualified_quality_phred 20 \
--length_required 20 \
--detect_adapter_for_pe
```

Contaminant removal: Univec (NOT DONE)
--------------------------------------

Screen for vector contamination using <a href="https://www.ncbi.nlm.nih.gov/tools/vecscreen/univec/" target="blank_">Univec</a>

``` bash
#### Index database #### 
bwa index UniVec 

#### map → sam → bam → filtered bam (unmapped) #### 
bwa mem 
-t 20 UniVec 
/shared/eng/water/Maria/UI_Sequencing/Pre_processing/01_fastp/$i/${i}.paired.R1.fq /shared/eng/water/Maria/UI_Sequencing/Pre_processing/01_fastp/$i/${i}.paired.R2.fq  | samtools view -hbS - | /home/opt/samtools-1.3.1/samtools view -hbF2 - > ${i}.bam 

#### sort bam #### 
samtools sort -n ${i}.bam -o ${i}.sorted

#### get fastq #### 
bedtools bamtofastq -i ${i}.sorted -fq ${i}.trimmed.filtered.R1.fq -fq2 ${i}.trimmed.filtered.R2.fq

#### Count reads #### 
awk 'END{printf ("%d\n", NR)}' ${i}.trimmed.filtered.R1.fq; cd ..; done >> R1_trimmed_filtered
awk 'END{printf ("%d\n", NR)}' ${i}.trimmed.filtered.R1.fq; cd ..; done >> R2_trimmed_filtered
```

Assembly or co-assembly
-----------------------

Check out Supplementary Table 1: <https://static-content.springer.com/esm/art%3A10.1038%2Fs41587-019-0191-2/MediaObjects/41587_2019_191_MOESM1_ESM.pdf> <https://www.nature.com/articles/s41587-019-0191-2>

Another assembler: <https://github.com/abishara/athena_meta> Athena is a read cloud assembler for metagenomes.

For purposes of hybrid assembly, a single short read assembly was performed first. <a href="https://github.com/voutcn/megahit" target="_blank">MegaHit</a> was used as assembler with default parameters and the --continue flag in case of interruptions due to server time constraint.

No preprocessing was performed in this instance:

``` bash
conda activate MEGAHIT_env

megahit -1 /scratch/m.sevi/processing/WW_HRSD/data/short/novaseq_run2/r_F02_1.fq.gz \
-2 /scratch/m.sevi/processing/WW_HRSD/data/short/novaseq_run2/r_F02_2.fq.gz \
-o /scratch/m.sevi/processing/WW_HRSD/hybrid_assembly/test1_random/RESULTS//intermediate_files/megahit_assembly \
--num-cpu-threads 10 --continue
```

Assembly quality
----------------

We can investigate assembly statistics to compare which assembly is best between the two assemblies utilized. For this we can use a software called Quast.

Metrics based only on contigs: Number of large contigs (i.e., longer than 500 bp) and total length of them. Length of the largest contig. N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly). Number of predicted genes, discovered either by GeneMark.hmm (for prokaryotes), GeneMark-ES or GlimmerHMM (for eukaryotes), or MetaGeneMark (for metagenomes).

Binning
-------

MaxBin uses both tetranucleotide frequencies and contig coverage levels to assign assembled contigs into different bins. It clusters whole contigs, which could contain regions common to multiple haplotypes and makes read coverage more heterogeneous. It is based on an Expectation-Maximization algorithm.

Followed this <a href="https://denbi-metagenomics-workshop.readthedocs.io/en/latest/binning/maxbin.html" target="_blank">MaxBin</a> tutorial.

### Generate an abundance file

#### Mapping with bwa

``` bash
conda activate bwa_env
bwa index /scratch/m.sevi/processing/WW_HRSD/02_single_assembly/short/r_F02/final.contigs.fa
bwa mem -t 20 /scratch/m.sevi/processing/WW_HRSD/02_single_assembly/short/r_F02/final.contigs.fa /scratch/m.sevi/processing/WW_HRSD/01_quality_control/QC/short/r_F02/r_F02_1.paired.R1.fq /scratch/m.sevi/processing/WW_HRSD/01_quality_control/QC/short/r_F02/r_F02_2.paired.R2.fq | samtools view -hbS - | samtools view -hbF4 - > short.bam 
samtools sort -n short.bam -o short.sorted
```

#### Create abundance file with bbmap's pileup

``` bash
pileup.sh in=/scratch/m.sevi/processing/WW_HRSD/02_single_assembly/short/mapping/short.bam  out=short.cov.txt
awk '{print $1"\t"$5}' short.cov.txt | grep -v '^#' > short.abundance.txt
```

### run MaxBin

This step is time consuming. Consider reducing the number of iterations. How to chose them?

``` bash
conda activate maxbin2_env

run_MaxBin.pl -thread 24 -contig /scratch/m.sevi/processing/WW_HRSD/02_single_assembly/short/r_F02/final.contigs.fa -out maxbin -abund short.abundance.txt
```

The output of this step is:

<table>
<thead>
<tr>
<th style="text-align:left;">
File
</th>
<th style="text-align:left;">
Description
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
(out).0XX.fasta
</td>
<td style="text-align:left;">
the.XX.bin..XX.are.numbers,.e.g..out.001.fasta
</td>
</tr>
<tr>
<td style="text-align:left;">
(out).summary
</td>
<td style="text-align:left;">
summary file describing which contigs are being classified into which bin.
</td>
</tr>
<tr>
<td style="text-align:left;">
(out).log
</td>
<td style="text-align:left;">
log file recording the core steps of MaxBin algorithm
</td>
</tr>
<tr>
<td style="text-align:left;">
(out).marker
</td>
<td style="text-align:left;">
marker gene presence numbers for each bin. This table is ready to be plotted by R or other 3rd-party software.
</td>
</tr>
<tr>
<td style="text-align:left;">
(out).marker.pdf
</td>
<td style="text-align:left;">
visualization of the marker gene presence numbers using R
</td>
</tr>
<tr>
<td style="text-align:left;">
(out).noclass
</td>
<td style="text-align:left;">
all sequences that pass the minimum length threshold but are not classified successfully.
</td>
</tr>
<tr>
<td style="text-align:left;">
(out).tooshort
</td>
<td style="text-align:left;">
all sequences that do not meet the minimum length threshold.
</td>
</tr>
</tbody>
</table>
More on binning: <http://merenlab.org/tutorials/infant-gut/#chapter-ii-automatic-binning>

MAG quality
-----------

<a href="" target="_blank">CheckM</a> allows us to assess the quality of genomes recovered from isolates, single cells, or metagenomes. It generates estimates for completeness and contamination by using collocated sets of genes that are ubiquitous and single-copy within a phylogenetic lineage.

CheckM also provides tools for identifying genome bins that are likely candidates for merging based on marker set compatibility, similarity in genomic characteristics, and proximity within a reference genome tree.

There are two general workflows: lineage and taxonomy based. As discussed in <a href="https://www.biostars.org/p/195935/" target="_blank">biostars</a>

Lineage (Phylogeny) workflow: use a set of markers that discriminate between phylogenetic groups. The markers chosen will be the most discriminating between groupings or placement on the phylogenetic tree.

Taxonomy workflow: their use of taxonomy is almost certainly the typical Kingdom Phylum... etc. You either generate a set of taxon-discriminant markers or generate a set of markers for a specific taxonomic group.

Ran the lineage workflow from CheckM. Step 1: \[CheckM - tree\] Placing bins in reference genome tree. Step 2: \[CheckM - lineage\_set\] Inferring lineage-specific marker sets. Step 3: \[CheckM - analyze\] Identifying marker genes in bins. Step 4: \[CheckM - qa\] Tabulating genome statistics.

``` bash
export PATH=/home/m.sevi/miniconda3/bin:$PATH
export PATH=/home/m.sevi/software/ANIcalculator_v1:$PATH

conda activate py2env

checkm lineage_wf . checkm_out \
-x fasta \
--threads 16 \
--pplacer_threads 16 \
--tmpdir checkm_tmp 
```

The qa output provides the following information:

<table>
<thead>
<tr>
<th style="text-align:left;">
Parameter
</th>
<th style="text-align:left;">
Description
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Bin Id
</td>
<td style="text-align:left;">
bin identifier (from input FASTA file)
</td>
</tr>
<tr>
<td style="text-align:left;">
Marker lineage
</td>
<td style="text-align:left;">
indicates lineage used for inferring marker set
</td>
</tr>
<tr>
<td style="text-align:left;">
Marker Id
</td>
<td style="text-align:left;">
UID indicates the branch within the reference tree used to infer the marker set applied to estimate the bins quality.
</td>
</tr>
<tr>
<td style="text-align:left;">
No. genomes
</td>
<td style="text-align:left;">
number of reference genomes used to infer marker set
</td>
</tr>
<tr>
<td style="text-align:left;">
No. markers
</td>
<td style="text-align:left;">
number of inferred marker genes
</td>
</tr>
<tr>
<td style="text-align:left;">
No. marker sets
</td>
<td style="text-align:left;">
number of inferred co-located marker sets
</td>
</tr>
<tr>
<td style="text-align:left;">
0-5+
</td>
<td style="text-align:left;">
number of times each marker gene is identified
</td>
</tr>
<tr>
<td style="text-align:left;">
Completeness
</td>
<td style="text-align:left;">
estimated completeness
</td>
</tr>
<tr>
<td style="text-align:left;">
Contamination
</td>
<td style="text-align:left;">
estimated contamination
</td>
</tr>
<tr>
<td style="text-align:left;">
Strain heterogeneity
</td>
<td style="text-align:left;">
estimated strain heterogeneity, determined from the number of multi-copy marker pairs which exceed a specified amino acid identity threshold (default = 90%). High strain heterogeneity suggests the majority of reported contamination is from one or more closely related organisms (i.e. potentially the same species), while low strain heterogeneity suggests the majority of contamination is from more phylogenetically diverse sources
</td>
</tr>
</tbody>
</table>
Refinement
----------

Want to use <a href="https://github.com/Vini2/GraphBin" target="_blank">GraphBin</a>, installation was easy, but application is a bit complicated

See issue: <https://github.com/Vini2/GraphBin/issues/4>

``` bash
<!-- ### https://github.com/voutcn/megahit -->
<!-- 1. get fastg from the intermediate contigs of k=141 -->
conda activate MEGAHIT_env
megahit_toolkit contig2fastg 141 out/intermediate_contigs/k141.contigs.fa > k141.fastg 

<!-- 2. fastg to gfa -->
fastg2gfa final.fastg > final.gfa

conda activate graphbin

python support/prepResult.py \
--binned /scratch/m.sevi/processing/WW_HRSD/02_single_assembly/short/maxbin2 \
--assembler MEGAHIT \
--output /scratch/m.sevi/processing/WW_HRSD/02_single_assembly/short/maxbin2/graphbin

python graphbin.py \
--assembler MEGAHIT \
--graph GRAPH         path to the assembly graph file\
--binned BINNED       path to the .csv file with the initial binning output from an existing tool\
--output OUTPUT       path to the output folder\

  
```
