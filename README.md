# Aligning DNA Sequencing Data Using BWA

Author: Wanjun Gu

Email: wagu@health.ucsd.edu



### Install Dependencies

```bash
sudo apt update -y;
sudo apt upgrade -y;
sudo apt install samtools -y;
sudo apt install seqtk -y;
```



### Setting up Burrows-Wheeler Aligner (BWA)

Download BWA from: http://sourceforge.net/projects/bio-bwa/files/

Once download, use:

```bash
tar xvfj bwa-*.tar.bz2 
cd bwa-*
make # Require cmake or other compilation tools installed already
export PATH=$PATH:/path/to/bwa-* # use export PATH=$PATH:~/bwa-* in wsl
source ~/.bashrc
```



### Downloading Reference Genome

```bash
# download hg19 chromosome fasta files
wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz

# unzip and concatenate chromosome and contig fasta files
tar zvfx chromFa.tar.gz
cat *.fa > hg19.fa
rm chr*.fa
```



### Create Reference

```bash
# bwa index [-a bwtsw|is] index_prefix reference.fasta
bwa index -p hg19bwaidx -a bwtsw hg19.fa

# -p index name (change this to whatever you want)
# -a index algorithm (bwtsw for long genomes and is for short genomes)
```



### Align to Reference Genome

```bash
# aligning single end reads
bwa aln -t 4 hg19bwaidx sequence1.fq.gz > sequence1.bwa
bwa samse hg19bwaidx sequence1.bwa sequence1.fq.gz> sequence1_se.sam

# aligning paired end reads
bwa aln -t 4 hg19bwaidx sequence1.fq.gz > sequence1.sai
bwa aln -t 4 hg19bwaidx sequence2.fq.gz > sequence2.sai
bwa sampe hg19bwaidx sequence1.sai sequence2.sai sequence1.fq.gz sequence2.fq.gz > sequence12_pe.sam
```



### Generate BAM files

```bash
samtools view -bT hg19.fa sequence1.sam > sequence1.bam # when no header
samtools view -bS sequence1.sam > sequence1.bam # when SAM header present

# sort by coordinate to streamline data processing
samtools sort -O bam -o sequence1.sorted.bam -T temp sequence1.bam 

# A position-sorted BAM file can also be indexed
samtools index sequence1.sorted.bam
```



### Converting the BAM file into fasta

```bash
samtools bam2fq input.bam | seqtk seq -A > output.fa
```



### Reference

Biostar Forum: https://www.biostars.org/p/129763/

Cambridge University Bioinformatics Slides: http://bioinformatics-core-shared-training.github.io/cruk-bioinf-sschool/Day1/Sequence%20Alignment_July2015_ShamithSamarajiwa.pdf