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
  
### [ ] Need to figure out a way to pass the enter key to the `sbfs configure` command bc it makes you press Enter twice, wants to confirm the APIURL and wants to confirm the developer token. 
https://stackoverflow.com/questions/6264596/simulating-enter-keypress-in-bash-script

- nothing from ^ page worked, I can still try the `expect` tool mentioned there but I need to get RIS to install it.

### [x] also need to figure out exactly how to pass args, (should just be 1-N where N is the # of splits) to the SLURM job array


If I cant mount my files to each of the worker nodes in the array, can I feed them downloadlinks and download the files on the worker nodes memory? BC if I do this in parallel then I wont have enough mem on the head node, if thats where downloaded data gets stored.



### [] Need to time the various steps on the cluster (norm, split, filter, sort, index)
- do I need to convert to bcf format first? Will that speed things up?

### Results of testing using this file, /vcf_test_path/006f8b8b-8d08-4cca-83a4-79c6a7195fac.g.vcf.gz (5.4GB) on Taylor cluster,:

Converting a full `.g.vcf` to `.bcf` with no filtering took: ~20min
`bcftools view --output-type b /home/stearb/gvcf_workflow/data/gvcfs/mount_dir/projects/taylordm/taylor-urbs-r03-kf-cardiac/vcf_test_path/006f8b8b-8d08-4cca-83a4-79c6a7195fac.g.vcf.gz  -o 006f8b8b-8d08-4cca-83a4-79c6a7195fac-SNPs.bcf.gz`

Same thing but with the SNPs flag so only SNPs are included took: ~7min
`bcftools view --types snps  --output-type b /home/stearb/gvcf_workflow/data/gvcfs/mount_dir/projects/taylordm/taylor-urbs-r03-kf-cardiac/vcf_test_path/006f8b8b-8d08-4cca-83a4-79c6a7195fac.g.vcf.gz  -o 006f8b8b-8d08-4cca-83a4-79c6a7195fac-SNPs.bcf.gz`

Norm/left-align and splitting multi-allelic sites from this `bcf` file took:
the -m flag will split multi-allelic sites, I specified only at SNP sites.
https://github.com/samtools/bcftools/issues/40
`bcftools norm -m snps -f FASTA.ref  `



BCFtools workflow plan:
1. Drop everything but SNPs and convert to .bcf using VIEW: `bcftools view --types snps  --output-type b in.vcf.gz -o out.bcf.gz`
2. Norm/left-align and split multi-allelic sites using NORM: `bcftools norm -m snps -f FASTA.ref`
   can also make index w/ norm ^^?
3. Sort using SORT
4. Index using INDEX
5. Merge using MERGE
   
   Do I need to resort/reindex/drop-duplicates after merging? 





