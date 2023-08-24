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

### Next step: download GATK 
https://gatk.broadinstitute.org/hc/en-us/articles/360041320571--How-to-Install-all-software-packages-required-to-follow-the-GATK-Best-Practices



# Can I mount the cavatica file system to the HPC and submit a SLURM job that uses cavatica files?


