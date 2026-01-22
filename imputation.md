ctrl+shift+v to preview in vs code  

# Quick guide:  
1. change build if necessary: liftoverplink.py(github)  
2. qc: https://www.well.ox.ac.uk/~wrayner/tools/  "Usage with 1000G reference panel"
3. 
```
#sort and bgzip
for i in *vcf  
do  
/data1/APPS/vcftools/src/perl/vcf-sort $i | bgzip > ${i}.gz  
done

#michigan api
TOKEN=<YOUR TOKEN>
curl https://imputationserver.sph.umich.edu/api/v2/jobs/submit/minimac4 \
  -H "X-Auth-Token: $TOKEN" \
  -F "files=@/mnt/data_schen_1/travis/thesis/target_samples/affy6/michigan_imputation/pre-imputation/affy6_liftOver_chr14.vcf.gz" \
  -F "refpanel=1000g-phase-3-v5" \
  -F "r2Filter=0.1" \
  -F "population=eur"
``` 
4. post_imputation  


## liftover:  
Genome build: must change to grch37(hg19) for imputation server.  

Check the genome build of the data in readme or sampleinfo file.   
Details about liftover see liftover protocol.  

```
python2 /mnt/data_schen_1/yimei/software_and_tools/liftover/liftOverPlink.py -m MAPFILE -p PEDFILE -o PREFIX -c CHAINFILE -e /mnt/data_schen_1/yimei/software_and_tools/liftover/liftOver 
plink --file <ped/map location(prefix)> --make-bed --out <output prefix> 
```
-m map file dir  
-p ped file dir  
-o prefix of output  
-c chain file(reference file)  
-e liftover software dir

## Use imputation preparation and checking tool by McCarthy group
reference: https://www.well.ox.ac.uk/~wrayner/tools/  
Tool location: /mnt/data_schen_1/yimei/software_and_tools/HRC-1000G-check-bim-v4.3.0/   
(Download HRC-1000G-check-bim-v4.2.zip from the reference website if you can't find it in the server)  
1. Create minor allele frequency report(.frq file)  
    - activate plink-env enrionment first if you havn't done so. It's needed for most of the time when you are doing imputation work.  
``` 
conda activate /mnt/data_schen_1/jingchun/miniconda3/envs/plink-env
plink --bfile <bfile location(prefix)> --freq --out <output prefix> 

plink --bfile /mnt/data_schen_1/jingchun/gwas_data/AD_GWAS/NG00043/NG00043_genotype_data_0/SYounkin_MayoGWAS_09-05-08_gds --freq --out SYounkin_MayoGWAS_09-05-08_gds
```
2. Run the tool:  
`perl /mnt/data_schen_1/yimei/software_and_tools/HRC-1000G-check-bim-v4.3.0/HRC-1000G-check-bim.pl -b <bim file dir> -f <frq file dir> -r <Reference panel dir> -g -p <population>` 

    - reference panel dir: /mnt/data_schen_1/yimei/software_and_tools/HRC-1000G-check-bim-v4.3.0/1000GP_Phase3_combined.legend  
    - population ussually is EUR. But sometimes we will get AFR or Asian data.  
1. The tool will generate a bash file at the current folder after the run. Run that bash file:  
`bash Run-plink.sh`
    
 ## Sort and bgzipped the qc-ed files
 ```
for i in *vcf  
do  
/data1/APPS/vcftools/src/perl/vcf-sort $i | bgzip > ${i}.gz  
done
```

 ## Upload the bgzipped files to the imputation server
michigan imputation server API: https://imputationserver.readthedocs.io/en/latest/api/  
We can use michigan imputation server API to upload
1. You need to register an account on the website. Then Get the token of your account.(The detailed guide is in the API reference page)
2. Upload the files. 
  - -F files line: the @ sign is necessary.
  - you can upload all chromosomes in one job.
```
#upload single file:
TOKEN=<your token>
curl https://imputationserver.sph.umich.edu/api/v2/jobs/submit/minimac4 \
  -H "X-Auth-Token: $TOKEN" \
  -F "files=@/mnt/data_schen_1/travis/thesis/target_samples/affy6/michigan_imputation/pre-imputation/affy6_liftOver_chr14.vcf.gz" \
  -F "refpanel=1000g-phase-3-v5" \
  -F "r2Filter=0.1" \
  -F "population=eur"
``` 
```
## upload multiple files
TOKEN=<your token>
curl https://imputationserver.sph.umich.edu/api/v2/jobs/submit/minimac4 \
  -H "X-Auth-Token: $TOKEN" \
  -F "files=@/mnt/data_schen_1/travis/thesis/target_samples/affy6/michigan_imputation/pre-imputation/affy6_liftOver_chr1.vcf.gz" \
  -F "files=@/mnt/data_schen_1/travis/thesis/target_samples/affy6/michigan_imputation/pre-imputation/affy6_liftOver_chr2.vcf.gz" \
  -F "files=@/mnt/data_schen_1/travis/thesis/target_samples/affy6/michigan_imputation/pre-imputation/affy6_liftOver_chr3.vcf.gz" \
  -F "refpanel=1000g-phase-3-v5" \
  -F "r2Filter=0.1" \
  -F "population=eur"
```
```
#self use. You don't have to do this way. 
#example for chr1-23. 
filedirprefix=/mnt/data_schen_1/jingchun/gwas_data/AD_GWAS/NG00023/imputation/preimputation/_adc2_complete.fwd.hg19_2018_1011
TOKEN=<your token>
curl https://imputationserver.sph.umich.edu/api/v2/jobs/submit/minimac4 \
  -H "X-Auth-Token: $TOKEN" \
  -F "files=@${filedirprefix}-updated-chr1.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr2.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr3.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr4.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr5.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr6.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr7.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr8.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr9.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr10.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr11.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr12.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr13.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr14.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr15.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr16.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr17.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr18.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr19.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr20.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr21.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr22.vcf.gz" \
  -F "files=@${filedirprefix}-updated-chr23.vcf.gz" \
  -F "refpanel=1000g-phase-3-v5" \
  -F "r2Filter=0.1" \
  -F "population=eur"
```

 ## Post-imputation work
 When the imputation is done, it will send an email. It contains the password to unzip the imputed data.
 1. go into the project folder(the folder where you want to store the results). create a folder for qc and log files. Download the qc and log files to that folder. (code is provided on the website.)  
      - cd to the log qc folder first and then use the command provided by the server to download all qc and log files.  
 2. create folder for each chromosome(chr1-22 usually. )
 ```
 #example code
 for ((i=1;i<=22;i++))
 do
 mkdir chr$i
 done
 ```
 3. create a folder for the merged results(for later use):   
 `mkdir merge`
 4. upload **update_name_v2_yl.py, post_imputation.bash** to the project folder. upload **mergelist_post_imputation.txt** in the merge folder you just created.
 5. download the imputed to the project folder
    - Recommend use screen to do this(detail about screen, see below or google) 
 6.  move zip file to each chromosome's folder and unzip.
 ```
#example code
#quotation mark in the next line is necessary
for ((i=1;i<=22;i++))
do
mv chr_${i}.zip chr${i}/
cd chr${i}
unzip -P "0Dj8DbcpTlnYN" chr_${i}.zip
cd ..
done
 ```
7. Run post_imputation.bash
    - This script is to convert imputed data back to plink binary format and removed all the indel location in the bim file for each chromosome. And it merge all chromosome's data together and do the qc.
    - make sure update_name_v2_yl.py are in the project folder.
    - Recommend to run the script in screen as well.
```
#bash post_imputation.bash <famfile location> <dataset name>
#example: 
bash post_imputation.bash  /mnt/data_schen_1/jingchun/gwas_data/AD_GWAS/NG00102/Genotype_data/SOMAtoShare_newID_parentID.fam SOMAtoShare_newID_parentID

for ((i=1;i<=22;i++))
do
cd chr${i}
unzip -P ":0Dj8DbcpTlnYN" chr_${i}.zip
cd ..
done
```
------------
**old imputation guide. can use for knowledge base for different tools but don't use it as reference for imputation procedures:**
 # Prepare for imputation

Tools might need:
-	Plink1.9: process binary data sets(fam,bed,bim)
-	LiftOver: update genome build: https://github.com/sritchie73/liftOverPlink
- screen(optional): run script or software at backend(so that we don't need to keep the ssh connection all the time)

Most of the softwares are installed in my conda environment. You need to activate it before starts everything:  
`conda activate /mnt/data_schen_1/jingchun/miniconda3/envs/plink-env`

Reference:  
plink: https://www.cog-genomics.org/plink/1.9/    
michigan imputation server: https://imputationserver.readthedocs.io/en/latest/  
screen: https://linuxize.com/post/how-to-use-linux-screen/  
vcf format: https://samtools.github.io/hts-specs/VCFv4.2.pdf


File types:  
- "bfile": plink binary file set. There are 3 files in the set:
  - .bed file: PLINK binary biallelic genotype table. Binary file, cannot be read.
  - .fam file: PLINK sample information file. Contain 6 columns. The columns we might use are:
    - col 1: Fam id(usually is group id or the sample id)
    - col 2: sample id
    - col 5: sex code('1' = male, '2' = female, '0' = unknown)
    - col 6: Phenotype value ('1' = control, '2' = case, '-9'/'0'/non-numeric = missing data if case/control)  
  Notes: if bfile is converted from vcf file. plink will separate the sample name by _ to get Fam id and sample id. For example, if the a sample name in vcf file is FAM_58C_WTCCC66375, fam id will be FAM_58C and sample id will be WTCCC66375. So make sure to replace any _ before convert them to bfile if _ doesn't separate fam id and sample id.
  - .bim file: PLINK extended MAP file. Contain 6 column:   
    - chromosome
    - variant id
    - centimorgan location(usually is 0 since it's rarely used)
    - base pair location
    - allele 1
    - allele 2.  
- vcf file and vcf.gz file: Variant calling format.  
Reference: https://samtools.github.io/hts-specs/VCFv4.2.pdf  
vcf file:
  - usually starts with information lines with "##" at the front  
```
##fileformat=VCFv4.2
##fileDate=20211230
##source=PLINKv1.90
```
  -  After the information lines is the header line. It contains 8 fixed columns:  
  1. #CHROM: chr number
  2. POS: bp position
  3. ID: variant id
  4. REF: reference allele
  5. ALT: alternative allele
  6. QUAL: quality of reads
  7. FILTER: filter status
  8. INFO: additional information.   
  If genotype data is present in the file, these are followed by a FORMAT column header, then an arbitrary number
of sample IDs.  
Example:  
```
#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  FAM_58C_WTCCC66061      FAM_58C_WTCCC66062
```
- After header, it's the variant information with one variant per line matched the header.  
vcf.gz: compressed vcf file. Recommend compressed vcf file through bgzip command instead of gzip(so that it can be used on bcftools.)

- tped,tfam,ped: PLINK data file. Can convert to bfile or vcf file with plink. We don't usually encounter it.  
 Details see: https://www.cog-genomics.org/plink/1.9/formats

