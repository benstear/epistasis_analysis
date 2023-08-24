# This page contains `bcftools` commands to filter, merge and analyze g.vcf files 





### Test out how long it takes to merge two full (no filtering done, no pre-built indices,each gvcf had N rows) g.vcf files

```
04a24c6b-a0c9-4edd-b339-67530fcc3039.g.vcf.gz
0562a0a1-76d8-494a-b368-8b2febfe206d.g.vcf.gz
```

- Using the CAVATICA app with this command it took `1h 43m` to complete 
`/opt/bcftools-1.15.1/bcftools merge  --output bcf_merge_2.merged.vcf --threads 12 --missing-to-ref --merge none --no-index --output-type v 005719a2-482f-4701-8f02-8b4bf9b84951.g.vcf.gz 006f8b8b-8d08-4cca-83a4-79c6a7195fac.g.vcf.gz`


On the Taylor cluster it took:


`bcftools merge  --output bcf_merge_FROM_TAYLOR_CLUSTER.merged.vcf --threads 12 --missing-to-ref --merge none --no-index --output-type v /home/stearb/gvcf_workflow/data/gvcfs/mount_dir/projects/taylordm/taylor-urbs-r03-kf-cardiac/04a24c6b-a0c9-4edd-b339-67530fcc3039.g.vcf.gz /home/stearb/gvcf_workflow/data/gvcfs/mount_dir/projects/taylordm/taylor-urbs-r03-kf-cardiac/0562a0a1-76d8-494a-b368-8b2febfe206d.g.vcf.gz`











