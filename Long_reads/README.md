Long read
================

Google Cloud
------------

Due to hardware contraints, we will be performing the basecalls on google cloud rather than Northeastern's compute cluster. The following instructions elaborate on Chris' prior work.

Download the <a href="https://cloud.google.com/sdk/docs#install_the_latest_cloud_tools_version_cloudsdk_current_version" target="_blank">gcloud CLI</a> to my local computer.

``` bash
<!-- #Use an older python version  -->
conda create -n gCloud python=2.7
conda activate gCloud 

python -V
<!-- # Python 2.7.18 :: Anaconda, Inc. -->

<!-- #Log in  -->
gcloud init --skip-diagnostics

<!-- #Create project -->
gcloud projects create ww-hrsd-nanopore

<!-- # Streamline screen reading property -->
gcloud config set accessibility/screen_reader true

<!-- #Set project  -->
gcloud config set project "ww-hrsd-nanopore"

<!-- # Check (https://cloud.google.com/sdk/docs/quickstart-macos) -->
gcloud info

<!-- #Create instance -->
<!-- #Note1: Free accounts have no GPU quotas, need to upgrade account and edit quotas -->
<!-- #Note2: Tried initially with nvidia-tesla-v100, but resource availability is an issue -->
gcloud compute instances create gpu-instance-1 \
    --machine-type n1-standard-8  \
    --zone us-west1-b \
    --accelerator type=nvidia-tesla-p100,count=1 \
    --boot-disk-size=750 \
    --tags http-server,https-server \
    --maintenance-policy=terminate \
    --no-restart-on-failure \
    --image-family tf-2-1-cu100 \
    --image-project deeplearning-platform-release \
    --metadata='install-nvidia-driver=True'

<!-- Created [https://www.googleapis.com/compute/v1/projects/ww-hrsd-nanopore/zones/us-west1-b/instances/gpu-instance-1]. -->
<!-- WARNING: Some requests generated warnings: -->
<!--  - Disk size: '750 GB' is larger than image size: '50 GB'. You might need to resize the root repartition manually if the operating system does not support automatic resizing. See https://cloud.google.com/compute/docs/disks/add-persistent-disk#resize_pd for details. -->

<!-- NAME            ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS -->
<!-- gpu-instance-1  us-west1-b  n1-standard-8               10.138.0.2   34.82.199.81  RUNNING -->

<!-- #SHH into instance -->
gcloud beta compute ssh --zone "us-west1-b" "gpu-instance-1" --project "ww-hrsd-nanopore"
```

Guppy Install
-------------

New 3.6.0 guppy.

``` bash
wget https://mirror.oxfordnanoportal.com/software/analysis/ont-guppy_3.6.0_linux64.tar.gz
tar -zxvf ont-guppy_3.6.0_linux64.tar.gz
```

Upload/Download data
--------------------

Data upload with scp is very time consuming. It took about 2hrs to copy 235G worth of data

``` bash
<!-- # Transfer with scp -->
scp -r m.sevi@xfer.discovery.neu.edu:/scratch/m.sevi/processing/WW_HRSD/data/long/first/VIP_NGS_RWI_091919/. .
```

Since scp is very time consuming, a good alternative is using <a href="https://cloud.google.com/compute/docs/instances/transfer-files#transfergcloud" target="_blank">cloud storage buckets</a>. To transfer data from the cluster to my instance I had to create an environment with the google cloud command line tool using conda:`conda install -c conda-forge google-cloud-sdk`. Then authenticated my account with `gcloud init --console-only` to avoid X11 forwarding issues. When doing this operation cluster --&gt; bucket --&gt; instance, I was able to transfer 155G of data in ~40 min.

``` bash
<!-- #Create bucket -->
gsutil mb -p ww-hrsd-nanopore -b on gs://long_read_fne/
gsutil mb -p ww-hrsd-nanopore -b on gs://last_eff/
gsutil mb -p ww-hrsd-nanopore -b on gs://nano_raw/

<!-- #Upload to bucket https://cloud.google.com/storage/docs/uploading-objects#gsutil -->
gsutil -m cp -r fne/VIP_NGS_Nano_WGS_2019_12_20/ gs://long_read_fne/
<!-- Operation completed over 1.0k objects/154.5 GiB.  -->
gsutil -m cp -r last_effluent/VIP_NGS_Nano_WGS_2019_10_02 gs://last_eff/
<!-- Operation completed over 1.8k objects/205.6 GiB.    -->
gsutil -m cp -r raw/VIP_NGS_Nano_WGS_2019_12_10/ gs://nano_raw/

<!-- #Download to instance https://cloud.google.com/storage/docs/downloading-objects-->
gsutil -m cp -r gs://long_read_fne/ fne/

<!-- <!-- #started @16:55- finished @17:20 --> -->
<!-- Operation completed over 1.0k objects/154.5 GiB.      -->

gsutil -m cp -r gs://last_eff/ last_eff/
gsutil -m cp -r gs://nano_raw/ raw/
Operation completed over 2.1k objects/282.9 GiB.   
```

Guppy
-----

Performing the actual basecalling for the 235G dataset took about Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 3.6.0+98ff765, client-server API version 1.1.0

``` bash
ont-guppy/bin/guppy_basecaller -i VIP_NGS_RWI_091919/VIP_RWI_091219/20190919_2025_MN30472_FAK72972_16a9214a/fast5/  -s out/first/first_fastq -c dna_r9.4.1_450bps_hac.cfg -x "cuda:0" --progress_stats_frequency 60 --compress_fastq --cpu_threads_per_caller=2


<!-- ONT Guppy basecalling software version 3.6.0+98ff765, client-server API version 1.1.0 -->
<!-- config file:        /home/m.sevillano/ont-guppy/data/dna_r9.4.1_450bps_hac.cfg -->
<!-- model file:         /home/m.sevillano/ont-guppy/data/template_r9.4.1_450bps_hac.jsn -->
<!-- input path:         VIP_NGS_RWI_091919/VIP_RWI_091219/20190919_2025_MN30472_FAK72972_16a9214a/fast5/ -->
<!-- save path:          out/first/first_fastq -->
<!-- chunk size:         2000 -->
<!-- chunks per runner:  512 -->
<!-- records per file:   4000 -->
<!-- fastq compression:  ON -->
<!-- num basecallers:    4 -->
<!-- gpu device:         cuda:0 -->
<!-- kernel path:         -->
<!-- runners per device: 4 -->

#This is confusing, there were 1950 raw fast5 files 
<!-- Found 1951 fast5 files to process. --> 
<!-- Init time: 8765 ms -->
<!-- [PROG_STAT_HDR] time elapsed(secs), time remaining (estimate), total reads processed, total reads (estimate), interval(secs), interval reads processed, interval bases processed -->

<!-- #started @ 13:46 PM finished 2:09 am next day : ~13 hrs -->

<!-- Caller time: 44603512 ms, Samples called: 162388447879, samples/s: 3.64071e+06 -->
<!-- There were fast5 file loading problems! Failed to load 1 out of 1951 fast5 files. Check log file for details. -->
<!-- Finishing up any open output files. -->
<!-- Basecalling completed successfully. -->


ont-guppy/bin/guppy_basecaller -i fne/long_read_fne/VIP_NGS_Nano_WGS_2019_12_20/VIP_NGS_FNE-1_112219/20191220_1836_MN30472_FAL12930_98e02ded/fast5/ -s out/fne/fne_fastq -c dna_r9.4.1_450bps_hac.cfg -x "cuda:0" --progress_stats_frequency 60 --compress_fastq --cpu_threads_per_caller=2

<!-- ONT Guppy basecalling software version 3.6.0+98ff765, client-server API version 1.1.0 -->
<!-- config file:        /home/m.sevillano/ont-guppy/data/dna_r9.4.1_450bps_hac.cfg -->
<!-- model file:         /home/m.sevillano/ont-guppy/data/template_r9.4.1_450bps_hac.jsn -->
<!-- input path:         fne/long_read_fne/VIP_NGS_Nano_WGS_2019_12_20/VIP_NGS_FNE-1_112219/20191220_1836_MN30472_FAL12930_98e02ded/fast5/ -->
<!-- save path:          out/fne/fne_fastq -->
<!-- chunk size:         2000 -->
<!-- chunks per runner:  512 -->
<!-- records per file:   4000 -->
<!-- fastq compression:  ON -->
<!-- num basecallers:    4 -->
<!-- gpu device:         cuda:0 -->
<!-- kernel path: -->
<!-- runners per device: 4 -->

<!-- Found 1014 fast5 files to process. -->
<!-- Init time: 5629 ms -->
<!-- [PROG_STAT_HDR] time elapsed(secs), time remaining (estimate), total reads processed, total reads (estimate), interval(secs), interval reads processed, interval bases processed -->

<!-- Caller time: 29351351 ms, Samples called: 112957211137, samples/s: 3.84845e+06 -->
<!-- Finishing up any open output files. -->
<!-- Basecalling completed successfully. -->

ont-guppy/bin/guppy_basecaller -i last_eff/last_eff/VIP_NGS_Nano_WGS_2019_10_02/VIP_FNE-091219/20191002_1916_MN30472_FAK82866_5c6c9be6/fast5/ -s out/last_eff/last_eff_fastq -c dna_r9.4.1_450bps_hac.cfg -x "cuda:0" --progress_stats_frequency 60 --compress_fastq --cpu_threads_per_caller=2

<!-- ONT Guppy basecalling software version 3.6.0+98ff765, client-server API version 1.1.0 -->
<!-- config file:        /home/m.sevillano/ont-guppy/data/dna_r9.4.1_450bps_hac.cfg -->
<!-- model file:         /home/m.sevillano/ont-guppy/data/template_r9.4.1_450bps_hac.jsn -->
<!-- input path:         last_eff/last_eff/VIP_NGS_Nano_WGS_2019_10_02/VIP_FNE-091219/20191002_1916_MN30472_FAK82866_5c6c9be6/fast5/ -->
<!-- save path:          out/last_eff/last_eff_fastq -->
<!-- chunk size:         2000 -->
<!-- chunks per runner:  512 -->
<!-- records per file:   4000 -->
<!-- fastq compression:  ON -->
<!-- num basecallers:    4 -->
<!-- gpu device:         cuda:0 -->
<!-- kernel path:         -->
<!-- runners per device: 4 -->

<!-- Found 1756 fast5 files to process. -->

<!-- Caller time: 50976173 ms, Samples called: 141453337631, samples/s: 2.77489e+06 -->
<!-- Finishing up any open output files. -->
<!-- Basecalling completed successfully. -->


ont-guppy/bin/guppy_basecaller -i raw/nano_raw/VIP_NGS_Nano_WGS_2019_12_10/VIP_NGS_RW-1_112019/20191210_2033_MN30472_FAK80197_fe25b131/fast5/ -s out/raw/raw_fastq -c dna_r9.4.1_450bps_hac.cfg -x "cuda:0" --progress_stats_frequency 60 --compress_fastq --cpu_threads_per_caller=2

<!-- ONT Guppy basecalling software version 3.6.0+98ff765, client-server API version 1.1.0 -->
<!-- config file:        /home/m.sevillano/ont-guppy/data/dna_r9.4.1_450bps_hac.cfg -->
<!-- model file:         /home/m.sevillano/ont-guppy/data/template_r9.4.1_450bps_hac.jsn -->
<!-- input path:         raw/nano_raw/VIP_NGS_Nano_WGS_2019_12_10/VIP_NGS_RW-1_112019/20191210_2033_MN30472_FAK80197_fe25b131/fast5/ -->
<!-- save path:          out/raw/raw_fastq -->
<!-- chunk size:         2000 -->
<!-- chunks per runner:  512 -->
<!-- records per file:   4000 -->
<!-- fastq compression:  ON -->
<!-- num basecallers:    4 -->
<!-- gpu device:         cuda:0 -->
<!-- kernel path:         -->
<!-- runners per device: 4 -->

<!-- Found 2138 fast5 files to process. -->

<!-- Caller time: 62166166 ms, Samples called: 200834764353, samples/s: 3.23061e+06 -->
<!-- Finishing up any open output files. -->
<!-- Basecalling completed successfully. -->
```

Transfer results back to discovery
----------------------------------

``` bash
scp -r first m.sevi@xfer.discovery.neu.edu:/scratch/m.sevi/processing/WW_HRSD/data/long/basecalled_fq/
scp -r fne m.sevi@xfer.discovery.neu.edu:/scratch/m.sevi/processing/WW_HRSD/data/long/basecalled_fq/
scp -r last_eff m.sevi@xfer.discovery.neu.edu:/scratch/m.sevi/processing/WW_HRSD/data/long/basecalled_fq/
scp -r raw m.sevi@xfer.discovery.neu.edu:/scratch/m.sevi/processing/WW_HRSD/data/long/basecalled_fq/
```

QC reads
--------

A tool called LongQC was used to evaluate the quality of the long reads. More information can be found in the author's <a href="https://github.com/yfukasawa/LongQC" target="_blank">github</a>

Another tool called NanoPlot was used. More info in the <a href="https://github.com/wdecoster/NanoPlot" target="_blank">github</a>.

``` bash
```

Pre processing
--------------

No pre-processing of long read data was performed. <https://github.com/marbl/canu/issues/1518> Option for read filtering: <https://github.com/rrwick/Filtlong>

Assembly
--------

Flye was used to perform single assembly of long reads. It is based on underlying repeat graphs data structure that tolerate more noise as opposed to DeBruijn graphs.

'Should I use raw or error-corrected reads?
Flye was designed and tested mainly using raw reads, so it is currently the recommended option.'
"How do I select genome size if I don't know it in advance? The genome size estimate is used for solid k-mer selection in the initial disjointig assembly stage. Flye is not very sensitive to this parameter, and the estimate could be rough. It is ok if the parameter is within 0.5x-2x of the actual genome size. If the final assembly size is very different from the initial guess, consider re-running the pipeline with an updated estimate for better results.

An alternative option is to run Flye in --meta mode, which uses a different approach for solid k-mer selection. This mode is almost independent from the genome size parameter (you still need to provide an estimate for the selection of some other parameters). When assembly is completed, you can re-run in the normal mode with the inferred genome size."

A comparative study of long reads assemblies reccommends Flye over Canu for **metagenomes**. Their benchmarking results can be found on their <a href="https://github.com/adlape95/Assembly-methods-nanopore" target="_blank">github</a>

Other options for assembly: \* pipeline: <https://www.forbes.com/sites/sergeiklebnikov/2020/06/17/20-year-old-robinhood-customer-dies-by-suicide-after-seeing-a-730000-negative-balance/#896d67816384>
