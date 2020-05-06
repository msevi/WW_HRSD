Long read pre-processing
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

New 3.6.0 guppy

``` bash
wget https://mirror.oxfordnanoportal.com/software/analysis/ont-guppy_3.6.0_linux64.tar.gz
tar -zxvf ont-guppy_3.6.0_linux64.tar.gz
```

Upload/Download data
--------------------

Data upload with scp is very time consuming.

``` bash
<!-- # Transfer with scp -->
scp -r m.sevi@xfer.discovery.neu.edu:/scratch/m.sevi/processing/WW_HRSD/data/long/first/VIP_NGS_RWI_091919/. .
```

To transfer data from the cluster to my instance I had to create an environment with the google cloud command line tool using conda:`conda install -c conda-forge google-cloud-sdk`. Then authenticated my account with `gcloud init --console-only` to avoid X11 forwarding issues. Since scp is very time consuming, a good alternative is using <a href="https://cloud.google.com/compute/docs/instances/transfer-files#transfergcloud" target="_blank">cloud storage buckets</a>

``` bash
<!-- #Create bucket -->
gsutil mb -p ww-hrsd-nanopore -b on gs://long_read_fne/

<!-- #Upload to bucket https://cloud.google.com/storage/docs/uploading-objects#gsutil -->
gsutil -m cp -r fne/VIP_NGS_Nano_WGS_2019_12_20/ gs://long_read_fne/

<!-- Operation completed over 1.0k objects/154.5 GiB.  -->

<!-- #Download to instance https://cloud.google.com/storage/docs/downloading-objects-->
gsutil -m cp -r gs://long_read_fne/ fne/

<!-- <!-- #started @16:55- finished @17:20 --> -->
<!-- Operation completed over 1.0k objects/154.5 GiB.      -->
```

Guppy
-----

Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 3.6.0+98ff765, client-server API version 1.1.0

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

<!-- Found 1951 fast5 files to process. -->
<!-- Init time: 8765 ms -->
<!-- [PROG_STAT_HDR] time elapsed(secs), time remaining (estimate), total reads processed, total reads (estimate), interval(secs), interval reads processed, interval bases processed -->

#started @ 13:46 PM
```
