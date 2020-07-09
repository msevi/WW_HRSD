Hybrid Assembly
================

Assembly pipelines
------------------

### MaSuRCA

Check out <a href="https://github.com/alekseyzimin/masurca" target="_blank">gitbhub</a> from developers

### Muffin

UNDER DEVELOPMENT <https://github.com/RVanDamme/MUFFIN>

### Opera MS

Check out <a href="https://github.com/CSB5/OPERA-MS" target="_blank">gitbhub</a> and <a href="https://www.nature.com/articles/s41587-019-0191-2" target="_blank">Nature paper</a>

#### Installation

<https://github.com/CSB5/OPERA-MS/issues/38>

``` bash
conda create -n OPERA-MS python=3.6
conda activate OPERA-MS
conda install -c bioconda perl-app-cpanminus
conda install -c r r-base
conda install -c anaconda gcc
export PERL5LIB=/home/m.sevi/miniconda3/envs/OPERA-MS/lib/5.26.2:$PERL5LIB
git clone https://github.com/CSB5/OPERA-MS.git
cd OPERA-MS
make
perl OPERA-MS.pl CHECK_DEPENDENCY

<!-- (OPERA-MS) [m.sevi@login-01 OPERA-MS]$ perl OPERA-MS.pl CHECK_DEPENDENCY -->
<!--  *** OPERA-LG functional -->
<!--  *** sigma functional -->
<!--  *** samtools functional -->
<!--  *** blasr functional -->
<!--  *** bwa functional -->
<!--  *** minimap2 functional -->
<!--  *** mash functional -->
<!--  *** mummer functional -->
<!--  *** megahit functional -->
<!--  *** spades functional -->
<!--  *** racon functional -->

<!-- pilon found in /scratch/m.sevi/processing/WW_HRSD/hybrid_assembly/OPERA-MS//utils/ is not functional. Checking path for pilon. -->
<!-- pilon not found in path. Exiting. -->

conda install -c bioconda pilon

<!-- The following packages will be downloaded: -->

<!--     package                    |            build -->
<!--     ---------------------------|----------------- -->
<!--     pilon-1.23                 |                2         6.6 MB  bioconda -->
<!--     r-base-3.2.2               |                0        18.9 MB -->
<!--     ------------------------------------------------------------ -->
<!--                                            Total:        25.4 MB -->

<!-- The following NEW packages will be INSTALLED: -->

<!--   libgcc             pkgs/main/linux-64::libgcc-7.2.0-h69d50b8_2 -->
<!--   openjdk            pkgs/main/linux-64::openjdk-8.0.152-h7b6447c_3 -->
<!--   pilon              bioconda/noarch::pilon-1.23-2 -->

<!-- The following packages will be REMOVED: -->

<!--   gfortran_impl_linux-64-7.3.0-hdf63c60_1 -->
<!--   gfortran_linux-64-7.3.0-h553295d_9 -->
<!--   gxx_impl_linux-64-7.3.0-hdf63c60_1 -->
<!--   gxx_linux-64-7.3.0-h553295d_9 -->
<!--   libgfortran-ng-7.3.0-hdf63c60_0 -->

<!-- The following packages will be SUPERSEDED by a higher-priority channel: -->

<!--   r-base                         r::r-base-3.6.1-haffb61f_2 --> pkgs/r::r-base-3.2.2-0 -->

perl OPERA-MS.pl CHECK_DEPENDENCY

<!-- (OPERA-MS) [m.sevi@login-01 OPERA-MS]$ perl OPERA-MS.pl CHECK_DEPENDENCY -->
<!--  *** OPERA-LG functional -->
<!--  *** sigma functional -->
<!--  *** samtools functional -->
<!--  *** blasr functional -->
<!--  *** bwa functional -->
<!--  *** minimap2 functional -->
<!--  *** mash functional -->
<!--  *** mummer functional -->
<!--  *** megahit functional -->
<!--  *** spades functional -->
<!--  *** racon functional -->
<!--  *** pilon functional -->

<!--  *** All compiled software are functional. -->
<!--  *** Please try to run OPERA-MS on the test dataset. -->

cd test_files
perl ../OPERA-MS.pl \
    --contig-file contigs.fasta \
    --short-read1 R1.fastq.gz \
    --short-read2 R2.fastq.gz \
    --long-read long_read.fastq \
    --out-dir RESULTS 2> log.err

<!-- This will assemble a low diversity mock community in the folder RESULTS. Note that in the case of interruption during an OPERA-MS run, using the same command-line will re-start the execution after the last completed checkpoint. -->


<!-- (OPERA-MS) [m.sevi@login-01 test_files]$ perl ../OPERA-MS.pl \ -->
<!-- >     --contig-file contigs.fasta \ -->
<!-- >     --short-read1 R1.fastq.gz \ -->
<!-- >     --short-read2 R2.fastq.gz \ -->
<!-- >     --long-read long_read.fastq \ -->
<!-- >     --out-dir RESULTS 2> log.err -->
<!--  *** First utilization: set-up of reference genome databases. Please wait ... -->
<!--  *** (1/3) Download of genomeDB_Sketch.msh -->
<!--  *** (2/3) Download of complete_ref_genomes.tar.gz -->
<!--  *** (3/3) Extraction of complete_ref_genomes.tar.gz -->

<!-- [Wed May 27 11:02:58 2020] Short read assembly [1/8] -->
<!-- [Wed May 27 11:02:58 2020] Skip [contig file provided as input] -->

<!-- [Wed May 27 11:02:58 2020] Long-read mapping and assembly graph generation [2/8] -->

<!-- [Wed May 27 11:06:57 2020] Short-read mapping and coverage estimation [3/8] -->

<!-- [Wed May 27 11:07:42 2020] Hierarchical clustering [4/8] -->

<!-- [Wed May 27 11:07:42 2020] Reference based clustering [5/8] -->

<!-- [Wed May 27 11:10:44 2020] Strain clustering and assembly [6/8] -->

<!-- [Wed May 27 11:14:45 2020] Assembly of other clusters [7/8] -->

<!-- [Wed May 27 11:14:46 2020] Gap filling [8/8] -->

<!-- [Wed May 27 11:15:45 2020] Assembly stats -->
<!-- Number of contigs: 484  -->
<!-- Assembly size: 9489737 bp -->
<!-- Max contig size: 3560270 bp -->
<!-- Contig(s) longer than 1Mbp: 2  -->
<!-- Contig(s) longer than 500kbp: 2 -->
<!-- Contig(s) longer than 100kbp: 12 -->
<!-- Contig N50: 1747745 bp -->


<!-- [Wed May 27 11:15:47 2020] OPERA-MS completed -->
```

The following checkpoints are completed in a run:
(1) Short read assembly
(2) Long-read mapping and assembly graph generation
(3) Hierarchical clustering
(4) Optimal clusters based on the BIC
(5) Mash genomic distance between each cluster and a database of complete bacterial genomes is computed
(6) Merge clusters (species-specific super-clusters)
(7) Deconvolute contigs that come from distinguishable subspecies genomes (subspecies level clustering)
(8) Scaffolding and gap-filling with OPERA-LG

In the case of interruption during a run, using the same command-line will re-start the execution after the last completed checkpoint.

The first checkpoint is not completed in time and it will start from scratch, as an alternative, short assembly was performed separately with MEGAHIT.

#### Usage (EDIT)

Scripts for MEGAHIT and OPERA-MS can be found in: `/scratch/m.sevi/processing/WW_HRSD/hybrid_assembly/test1_random`

### Evaluate preformance

I'll evaluate the performance using metaquast based on single short read assembly, single long read assembly, and hybrid assembly. Pre-processing of short read data and its effect on single short read assembly and hybrid assembly is also evaluated. The tool used to perform this comparison is MetaQuast

<http://quast.sourceforge.net/metaquast>

<https://github.com/ablab/quast>
