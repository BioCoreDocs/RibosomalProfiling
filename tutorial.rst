===================================
**Ribofilio Quick Tutorial**
===================================



Download Data
##################

Prepare a directory:: 

       mkdir ribosomal 
       cd ribosomal 

Let's donwload a sample of yeast ribosomal data:: 

    curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR594/009/SRR5945809/SRR5945809.fastq.gz
    curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR594/008/SRR5945808/SRR5945808.fastq.gz


Trimming adapters and Quality Filtering
###########################################

We will create two folders for galore and fastq:: 
    
    mkdir galore 
    mkdir fastqc 

Then we use trim_galore to autodetetc and trim adaptors and quality filtering:: 

    trim_galore --gzip --retain_unpaired --trim1 --fastqc --fastqc_args "--outdir fastqc" -o galore SRR5945808.fastq.gz 
    trim_galore --gzip --retain_unpaired --trim1 --fastqc --fastqc_args "--outdir fastqc" -o galore SRR5945809.fastq.gz

Alignment to Reference
###########################

Let's first download the yeast reference transcriptome:: 

    curl -O ftp://ftp.ensembl.org/pub/release-98/fasta/saccharomyces_cerevisiae/cdna/Saccharomyces_cerevisiae.R64-1-1.cdna.all.fa.gz 
    gunzip Saccharomyces_cerevisiae.R64-1-1.cdna.all.fa.gz
    mv  Saccharomyces_cerevisiae.R64-1-1.cdna.all.fa yeast.fa 


Then we build a bowtie2 index for yeast:: 
  
    bowtie2-build  yeast.fa yeast  
    

We will use bowtie2 for alignment:: 

    bowtie2 -x yeast -U galore/SRR5945808_trimmed.fq.gz -S SRR5945808.bowtie2.sam
    bowtie2 -x yeast -U galore/SRR5945808_trimmed.fq.gz -S SRR5945809.bowtie2.sam  


And convert sam to a bam file::
 
    samtools view -S -b SRR5945808.bowtie2.sam > SRR5945808.bowtie2.bam
    samtools view -S -b SRR5945809.bowtie2.sam > SRR5945809.bowtie2.bam


Then convert bam to bed files for ribofilio:: 


    bedtools bamtobed -i SRR5945808.bowtie2.bam > SRR5945808.bed
    bedtools bamtobed -i SRR5945809.bowtie2.bam > SRR5945809.bed 

Then run ribosilio as follows:: 


   python ribofilio.py -t yeast.fa -f SRR5945809.bed -r SRR5945808.bed -b 50 -c 10


The above command runs ribofilio on the footprint sample SRR5945809.bed and normalize it with the mRNA sample SRR5945808.bed, using binsize =50 and minimum 10 genes as a cutoff for number of genes supporting a position in the bin. 




