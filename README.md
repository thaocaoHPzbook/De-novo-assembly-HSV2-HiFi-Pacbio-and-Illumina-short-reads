# De-novo-assembly-HSV2-HiFi-Pacbio-and-Illumina-short-reads    
Publication https://pmc.ncbi.nlm.nih.gov/articles/PMC8461477/    
Data download from ENA https://www.ebi.ac.uk/ena/browser/view/PRJEB40042 (data HiFi Pacbio long reads: ERR3278853; Data Illumina short reads: ERR3278849_1.fastq.gz and ERR3278849_2.fastq.gz)

## Download data    
**1. Pacbio HiFi data**    
```bash
mkdir HSV2_HiFi
da HSV2_HiFi
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR327/009/ERR3278853
gunzip ERR3278853.fastq.gz
```

**2. Illumina data**    
```bash
mkdir HSV2_Illumina
cd HSV2_Illumina
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR327/009/ERR3278849/ERR3278849_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR327/009/ERR3278849/ERR3278849_1.fastq.gz
gunzip ERR3278849_2.fastq.gz
gunzip ERR3278849_1.fastq.gz
```
## Quality check the downloaded data    
**1. Install fastqc**    
```bash
conda create -n fastqc_env
conda activate fastqc_env
conda install -c bioconda fastqc
```
**2. Run fastqc for Pacbio HiFi reads**    
```bash
cd HSV2_HiFi
mkdir -p fastqc_output
conda activate fastqc_env
fastqc ERR3278853.fastq.gz -o fastqc_output
```

**3. Run fastqc for Illumina reads**    
```bash
cd HSV2_Illumina
mkdir -p fastqc_output
conda activate fastqc_env
fastqc ERR3278849_1.fastq ERR3278849_2.fastq -o fastqc_output
```
## De novo assembly HiFi reads data with canu tool    
**1. Install canu**    
```bash
conda create -n canu_2_2_env
conda activate canu_2_2_env
conda install -c bioconda canu
```
**2. Run canu**    
```bash
conda activate canu_2_2_env
canu -p hsv -d hsv_assembly genomeSize=155k maxThreads=8 -ERR3278853.fastq
```
Output directory named **hsv_assembly** will be created automatically, containing de novo assembly results


