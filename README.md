
## Denovo Assembly for Long Read Sequences

Long read sequencing technologies such as PacBio and Oxford Nanopore are becoming more popular in the field of genomics. These technologies are capable of generating long reads. The long reads are useful for assembling the genome of complex organisms. In this project, we will use the PacBio HiFi reads to assemble the genome of yeast Saccharomyces cerevisiae.


> "Software installation is like a box of chocolates, you never know what you're gonna get. No matter what tool you use, conda, mamba, sudo, pip,... it always finds a way to throw an error and refuse to install.
>
> ---- A frustrated bioinformatician


**Data**

I used the filtered and clipped version of fastq file for the analysis [(link)](https://trace.ncbi.nlm.nih.gov/Traces/?view=run_browser&acc=SRR13577846&display=download)

For simplicity, I just put all the inputs in **../data/** folder to make `git push` easier (we could use .gitignore too). Don't be worry, everytime we need an input, I will mention the url.


**1 - Quality Control of the reads**

sofware: FastQC v0.11.9

The first step is to check the quality of the reads. We can use FastQC to do this. The command is given below:

```console

$  fastqc  <input>  -o <output>

```

```console

$  fastqc  ../data/SRR13577846.fastq.gz  -o 1-QC/


$  firefox 1-QC/SRR13577846_fastqc.html  # open the report in firefox

```

There are some alert in fastqc report. We can ignore them for now as it is an timely intensive excerise. Maybe we will come back to this later.


**2 - Perform de novo assembly using Hifiasm**

software: Hifiasm 0.18.8-r525 [(Link)](https://hifiasm.readthedocs.io/en/latest/index.html)

Hifiasm is a fast and accurate assembler for PacBio HiFi reads. The command is given below:

```console

hifiasm -o <output> -t <threads> <input>

```

```console

$ hifiasm -o 2_assembly/SRR13577846 -t 5 ../data/SRR13577846.fastq.gz

# Real time: 1602.548 sec; CPU: 7809.897 sec; Peak RSS: 13.164 GB

```

hifiasm output is a set of files. You can find the details in the documentation. We us `<>.bp.p_ctg.gfa` file which contains the assembled contigs.


Quast and BUSCO needs contigs in fasta format. We can use `awk` to convert the gfa file to fasta file.

```console

$ awk '/^S/{print ">"$2;print$3}' 2_assembly/SRR13577846.bp.p_ctg.gfa > 2_assembly/SRR13577846.fa

```

The fasta file "SRR13577846.fa" is the input for Quast and BUSCO.


**3 - Perform quality assessment using Quast**

software: Quast v5.0.2 [(link)](https://quast.bioinf.spbau.ru/)

Quast is a tool for quality assessment of genome assemblies. The command is given below:

Saccharomyces cerevisiae reference genome is available at [this page](https://www.ncbi.nlm.nih.gov/genome/?term=Saccharomyces%20cerevisiae%5BOrganism%5D&cmd=DetailsSearch#:~:text=This%20single%2Dcelled%20organism%20is,Mb%2C%20organized%20in%2016%20chromosomes.). You can [download this](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_genomic.fna.gz) and put it in `../data/` folder. I renamed it to `ref.fna`.

```console

$ quast -r <reference> <input> -o <output>

```

```console

$ quast -r ../data/ref.fna 2_assembly/SRR13577846.fa -o 3_QUAST/

$ firefox 3_QUAST/report.html

```


**4 - Perform quality assessment using BUSCO**

software: BUSCO v4.1.4 [(link)](https://busco.ezlab.org/)

BUSCO is a tool for quality assessment of genome assemblies which is based on the presence of orthologous genes.

```text
busco --list-datasets # to find lineage also can be selected autolineage
```

 The command is given below:

```console

$ run_BUSCO.py -i <input> -o <output> -l <lineage> -m <mode> -c <threads>

```

```console

$ run_BUSCO.py -i 2_assembly/SRR13577846.fa -o 4_BUSCO/SRR13577846 -l saccharomycetes_odb10 -m genome -c 5

```
