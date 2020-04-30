Short read pre-processing
================

Renaming
--------

FastP
-----

``` bash
####  Create new environment #### 
conda create -n fastp_env

####  Activate environment and install fastp #### 
conda activate fastp_env
conda install -c bioconda fastp

#### Script #### 
#!/bin/sh
#SBATCH --array=1-3
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=20
#SBATCH --time=24:00:00
#SBATCH --mem=250000
#SBATCH --job-name=CARD
#SBATCH --error=job.%a_%x_%j.stderr
#SBATCH --output=job.%a_%x_%j.stdout
# --dependency=singleton
#SBATCH --partition=short

source ~/miniconda3/bin/activate
conda activate fastp_env
fastp --thread 20 
--in1 
--out1  
--in2 
--out2 
--trim_poly_g 
--trim_poly_x 
--qualified_quality_phred 20 
--length_required 20 
--adapter_fasta  specify a FASTA file to trim both read1 and read2 (if PE) by all the sequences in this FASTA file (string [=])
```

Contaminant removal: Univec
---------------------------

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