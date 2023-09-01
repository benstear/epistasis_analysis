# This page describes the process of analyzing the gVCFs from the Kids First CHD cohort.
and progress of transfering the gVCF files (in .g.vcf or .parquet format) and analyzing them on the Taylor Lab server

First ssh onto the remote server by doing: `ssh RESLNTAYLORD01.research.chop.edu`

This part of the workflow is in `/home/stearb/gvcf_workflow`
### Aug 2023
Just mount the CAVATICA project

#### Data Transfer notes:
- Either get download links from Files tab on Cavatica or use sbfs cli (transfer parquet or g/vcf? Can’t get download link for parquet bc its a directory)

A .g.vcf file of about 7.8GB takes ~6min to transfer to the cluster using:
`curl $(cat test_gvcf_download_links2.txt) --output first_gvcf.g.vcf`

To check what the file looks like, do: `zcat first_gvcf.g.vcf | head -n 10000`

test_gvcf_download_links2.txt is a list of download links


**Test if mounting the project files with `sbfs` is faster…**

To download `sbfs cli`, do: `sudo yum install fuse`   # need to have fuse installed first
`curl https://igor.sbgenomics.com/downloads/sbfs/install.sh -sSf | sudo sh`       # from https://docs.sevenbridges.com/docs/install-sbfs-automatically
sbfs configure

Use this command to download sbfs (from this [page](https://docs.sevenbridges.com/docs/command-line-interface)):

`bash -c 'curl https://igor.sbgenomics.com/downloads/sb/install.sh -sSf ø sudo -H sh'`

Do NOT use the command on the CAVATICA sbfs install page

`sb configure`
Follow directions on this page: https://docs.sevenbridges.com/docs/store-credentials-to-access-seven-bridges-client-applications-and-libraries

`mkdir .sevenbridges; cd .sevenbridges; vi credentials; mv .sevenbridges/* /home/stearb`

```
[taylordm]
api_endpoint = https://cavatica-api.sbgenomics.com/v2
auth_token = c05edbf9ffe041479f063666e23f675f
```

 --------------------------------
 
Then mount the CAVATICA project

Command Signature: `sbfs mount ~/documents/my-folder --project rfranklin/my-project`

To mount the project: 

```
cd /home/stearb/gvcf_workflow/data/gvcfs/; mkdir mount_dir
sbfs mount mount_dir/ --project taylordm/taylor-urbs-r03-kf-cardiac
mount_dir/projects/taylordm/taylor-urbs-r03-kf-cardiac
```

To unmount:  `sbfs unmount mount_dir/`

### I will just mount my project so I have access to the files w/o having to transfer and store them on the remote server
### I mounted the project in mount_dir in /home/stearb/gvcf_workflow/data/gvcfs/mount_dir/projects/taylordm/taylor-urbs-r03-kf-cardiac


### Next step: download bcftools (http://samtools.github.io/bcftools/howtos/install.html go to)
```
git clone --recurse-submodules https://github.com/samtools/htslib.git
git clone https://github.com/samtools/bcftools.git
cd bcftools
autoheader && autoconf && ./configure --enable-libgsl --enable-perl-filters
autoreconf
sudo make install
which bcftools
```

### Next step: download GATK <-----NOT YET...
https://gatk.broadinstitute.org/hc/en-us/articles/360041320571--How-to-Install-all-software-packages-required-to-follow-the-GATK-Best-Practices

How to submit multiple jobs using parameters with a Python helper script.
https://www.osc.edu/book/export/html/4046


#### Can I mount the cavatica file system to the HPC and submit a SLURM job that uses cavatica files?

-----------------

# September 1st update

### [x] Got RIS to install `sbfs` on Respublica HPC
### [x] Made a file that contains `N` gVCF filenames (of the 711 probands from the CHD) per row so that I can feed it to a job or a script, and each line will be fed to a different thread/task/CPU. So if each line contains 10 file names, then I'll have about 71 tasks, running in parallel, each of which will be emerging 10 gVCFs.

Also need to have a file that contains a simple list of the file names so that I can run the following commands on every file:
- `Norm/left align`
- `split multi-allelic sites`
- `filter` (what filters — )
- `sort`
- `index`
  
### [] Need to figure out a way to pass the enter key to the `sbfs configure` command bc it makes you press Enter twice, wants to confirm the APIURL and wants to confirm the developer token. 
https://stackoverflow.com/questions/6264596/simulating-enter-keypress-in-bash-script


### [] also need to figure out exactly how to pass args, (should just be 1-N where N is the # of splits) to the SLURM job array


if I have a script like so:

SBATCH -array 1-N
SBATCH mem-
SBTATCH time-
SBATCH name-
module load sbfs
module load BFCtools
sbfs mount ...
bcftools merge $SLURM_ARRAY_TASK_ID

need to somehow pass this as an arg to index the file of filenames 




