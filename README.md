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

## Quality check after de novo assembly    
**1. Install QUAST tool**   
```bash
conda create -n quast_env
conda activate quast_env
conda install -c bioconda quast
```
**2. Run QUAST**    
```bash
cd hsv_assembly
quast.py -o quast_output hsv.contigs.fasta
```

### Scaffolding with RaGOO
**1. Install RaGOO**
```bash
git clone https://github.com/schatzlab/ragoo.git
cd ragoo
conda create -n ragoo_env python=3
conda activate ragoo_env
conda install numpy networkx
```

**2. Download HSV2 reference genome**
```bash
wget https://path_to_genome/MK855052.fasta
mv MK855052.fasta reference.fasta
```

**3. Run RaGOO**
```bash
ragoo.py hsv.contigs.fasta reference.fasta
```
**ragoo_output** folder is genernerated automatically, containing the scaffolding results.   

## Quality check after scaffolding
```bash
cd ragoo_output
quast.py -o quast_output ragoo.fasta
```

## Polising the scaffold by Illumina reads with Pilon tools
**1. Install bwa, samtools and pilon**
```bash
conda create -n bwa_env python=3.8
conda activate bwa_env
conda install -c bioconda bwa
```
```bash
conda create -n samtools_env python=3.8
conda activate samtools_env
conda install -c bioconda samtools
```
```bash
conda create -n pilon_env python=3.8
conda activate pilon_env
conda install -c bioconda pilon
```

**2. Run BWA, samtools and pilon**
```bash
conda activate bwa
bwa index ragoo.fasta
bwa mem ragoo.fasta ERR3278849_1.fastq ERR3278849_2.fastq > aligned_reads.sam
conda deactivate
```
```bash
conda activae samtools_env
samtools view -bS aligned_reads.sam > aligned_reads.bam
samtools sort aligned_reads.bam -o aligned_reads_sorted.bam
samtools index aligned_reads_sorted.bam
conda deactivate
```
```bash
conda activate pilon_env
java -Xmx16g -jar /home/hp/miniconda3/envs/pilon_env/share/pilon-1.24-0/pilon.jar --genome ragoo.fasta --frags aligned_reads_sorted.bam --output pilon_polished --changes
conda deactivate
```

##  Quality check after polishing
```bash
quast.py -o quast_output_pilon pilon_polished.fasta
```

## Genome annotation with prokka
**1. Install prokka**
```bash
conda create -n prokka_env python=3.8
conda activate prokka_env
conda install -c bioconda prokka
```

**2. Run prokka**
```bash
conda activate prokka_env
prokka --outdir prokka_hsv2 --prefix hsv2 \
  --kingdom Viruses --genus Herpesvirus --species "Human herpesvirus 2" \
  --addgenes --metagenome --force \
  --cpus 4 \
 pilon_polished.fasta
```









