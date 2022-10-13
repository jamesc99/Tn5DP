# Tn5DP (Tn5-based epigenetic Data Processing)
Pipeline for the QC metrics construction, data analysis and visualization of ATAC-seq and CUT&Tag-seq data. 

Current Version: `Tn5DP_v1.0` Last update: `2022.10.12`


Advisor: Bo Zhang<br/>Contributor: Siyuan Cheng

For any question, please contact siyuancheng@wustl.edu :point_left:

<br />
<br /> 

## Documentation
1. Pipeline documentation: analysis details, parameters, and QC metrics information<br/>Please [click here](documents/Documentation.md)
2. Update logfile: pipeline change record<br/>Please [click here](documents/update_log.md)
3. Check the definition of narrow/broad marker, and mapped-read numbers<br/>Please [click here](https://www.encodeproject.org/chip-seq/histone/)

<br />
<br />

## Usage:
### Singularity3 Installation
Tn5DP image has to be run with **Singularity version3+**, you could follow this instruction if you haven`t install Singularity3. <br/>Please [click here](https://github.com/sylabs/singularity/blob/main/INSTALL.md)<br/>(You will need sudo permission to properlly install and configure it, but you can run it without sudo after installation:smiley:)

### Test ATAC-seq data:
There are one paired-end mm10 data with 0.25M reads for test purpose, they can be downloaded by:
```
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/sample_data/atac-seq/mm10_1.fastq.gz
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/sample_data/atac-seq/mm10_2.fastq.gz
```

### Test CUT&Tag-seq data:
There are one paired-end mm10 data (H3K36me3 marker) with 0.25M reads for test purpose, they can be downloaded by:
```
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/sample_data/cuttag-seq/mm10_k36me3_1.fastq.gz
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/sample_data/cuttag-seq/mm10_k36me3_2.fastq.gz
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/sample_data/cuttag-seq/mm10_igg_1.fastq.gz
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/sample_data/cuttag-seq/mm10_igg_2.fastq.gz
```

### Run Tn5DP:
**Step1** Download the singularity image and reference files (you only need download them **ONCE**, then you can use them directly), if there is any update, you may need to download a new image, but reference files are usually **NOT** changed:

1. Download the singularity image:
```
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/Tn5DP_v1.0.simg
```

2. Download the reference files of different genome:
```
wget http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/Genome/mm10.tar.gz
```
You can also find more genome builds: [click here](http://regmedsrv1.wustl.edu/Public_SPACE/ryan/Public_html/singularity_ac/Genome/) . Currently we have: mm9/10/39, hg19/38, danRer10/11, rn6 and dm6.

3. Decompress the reference files and put to your own folder:
```
tar -zxvf mm10.tar.gz
```

**Step2** Process data by the singularity image:

**:bangbang:Please run the cmd on the same directory of your data, if your data is on /home/example, then you may need `cd /home/example` first. The location of image and reference files is up to you.**
```
singularity run -B ./:/home -B <path-to-parent-folder-of-ref-file>:/cuttag_atac/Resource/Genome <path-to-downloaded-image> \
-d <ATAC/CUTnTag> -g <hg38/mm10 etc.> -r <PE/SE> -m <narrow/broad> \
-o <experimental read file1>  -O <experimental read file2> \
-i  <IgG_control read file1>  -I <IgG_control read file2> \
-s  <AIAP/MACS2/F-seq/SEACR/HMMRATAC/ChromHMM> \
--personalize (use this option when using customized peakcalling parameter setting, default is False) \
--peakcalling options (if you use --personalize option)
```
For example, if<br/>a) you download the image on /home/image/Tn5DP_v1.0.simg<br/>b) the reference file on /home/src/mm10<br/>c) and your data type is ATAC-seq data<br/>d) and experiment data is read1.fastq.gz and read2.fastq.gz on folder /home/data<br/>e) and input data is igg_1.fastq.gz and igg_2.fastq.gz on folder /home/data<br/>f) and you want to run the recommended pipeline for ATAC-seq data

Then you need to:
1. `cd /home/data`
2. `singularity run -B ./:/home -B /home/src:/cuttag_atac/Resource/Genome /home/image/Tn5DP_v1.0.simg -d ATAC -g mm10 -r PE -o read1.fastq.gz -O read2.fastq.gz -i igg_1.fastq.gz -I igg_2.fastq.gz -s AIAP`

Note: <br/>for ATAC-seq data, we recommend AIAP for peak calling,<br/>for CUT&Tag-seq narrow marker data, we recommend MACS2 for peak calling,<br/>for CUT&Tag-seq broad marker data, we recommend ChromHMM for peak calling<br/>for the details of **personalized peak calling options**, you can [check here](documents/Documentation.md)

**Step3** Differential analysis (if needed)

  We use **edgeR** and peakcalling results, as well as BED files generated by Tn5BP to do DBR (Differeital Binding Regions) analysis. We support the user-defined value for CPM, q-value, and log2FC parameters
```
# similar to DEG analysis, you will need 2 conditions to compare, and sample size for each condition is at least 2, unbalanced design is supported.
# please put read file (step3.3*open.bed in ATAC-seq and step3.1*.extended.bed in CUT&Tag-seq) and peak file (*step4.1_peakcall_*) of the compared data into same folder
# please make sure for each condition, there is a keyword to group them together, namely, use ls *keyword1* can catch all read and peak files in group1, and ls *keyword2* can catch all read and peak files in group2

# run DBR analysis
singularity exec <path-2-singularity-image> bash /cuttag_atac/Resource/pipe_code/DBR_analysis.sh keyword1 keyword2 cpm_value q_value log2FC_value
```
For example, after running Tn5DP: <br/>a) create a folder for DBR analysis `mkdir /home/dbr_analysis` <br/> b) copy all `step4.1*` files to /home/dbr_analysis <br/>c) Prepare read bed files (step3.3*open.bed in ATAC-seq and step3.1*.extended.bed in CUT&Tag-seq) <br/> d) find the keywords for 2 conditions. Let`s assume the keywords for two groups will be *d0* and *d2* <br/> e) choose CPM_value, q-value, and Log2FC_value. Let`s assume you deem them as 1, 0.05, 1.5

Then you need to:
1. `cd /home/dbr_analysis`
2. `singularity exec /path-to-image/Tn5DP_v1.0.simg bash /cuttag_atac/Resource/pipe_code/DBR_analysis.sh d0 d2 1 0.05 1.5
`


























