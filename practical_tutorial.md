# A practical tutorial for genome assembly

## Table of Contents
[Part 1 - Estimating genome size](#part-1---estimating-genome-size) &nbsp;&nbsp;&nbsp;&nbsp; <ins>[Quiz](#quiz)</ins>\
[Part 2 - Genome assembly and evaluation](#part-2---genome-assembly-and-evaluation)\
&nbsp;&nbsp;&nbsp;&nbsp; [2.1 - Genome assembly with hifiasm](#21---genome-assembly-with-hifiasm)\
&nbsp;&nbsp;&nbsp;&nbsp; [2.2 - Running BUSCO](#22---running-busco) &nbsp;&nbsp;&nbsp;&nbsp; <ins>[Quiz](#quiz-1)</ins>\
&nbsp;&nbsp;&nbsp;&nbsp; [2.3 - Interpreting purge_dups outputs](#23---interpreting-purge_dups-outputs)\
&nbsp;&nbsp;&nbsp;&nbsp; [2.4 - Kmer plots to evaluate assembly quality](#24---kmer-plots-to-evaluate-assembly-quality)\
&nbsp;&nbsp;&nbsp;&nbsp; [2.5 - Evaluating pre and post genome purging](#25---evaluating-pre-and-post-genome-purging) &nbsp;&nbsp;&nbsp;&nbsp; <ins>[Quiz](#quiz-2)</ins>\
[Part 3 - Polishing assemblies](#part-3---polishing-assemblies)\
[Part 4 - Scaffolding assembly with Hi-C data](#part-4---scaffolding-assembly-with-hi-c-data) &nbsp;&nbsp;&nbsp;&nbsp; <ins>[Quiz](#quiz-3)</ins>\
[Part 5 - Quiz and presentation](#part-5---quiz-and-presentation) &nbsp;&nbsp;&nbsp;&nbsp; <ins>[Quiz](#part-5---quiz-and-presentation)</ins>

***
Consider the two species in the following paths and choose the one you will work with. The species are [Eristalis arbustorum](https://tolqc.cog.sanger.ac.uk/darwin/insects/Eristalis_arbustorum/) (ToLID: **idEriArbu1**) and [Thymelicus sylvestris](https://tolqc.cog.sanger.ac.uk/darwin/insects/Thymelicus_sylvestris/) (ToLID: **ilThySylv1**). The genomic data for them are here:
  
    /home/training/assembly/data/assembly/idEriArbu1
    /home/training/assembly/data/assembly/ilThySylv1

Create a working folder for this course:

    rm -r /home/training/assembly/workdir
    mkdir /home/training/assembly/workdir

## Part 1 - Estimating genome size

As you learned in the theoretical lectures, genome sizes can be estimated by kmer analyses of high-quality reads, such as Illumina and Pacbio HiFi. To do it so, first you would need install kmc on your computer and generate a kmer count with it. You can have a look at the example command line below although we will not run KMC in this course and you **don't have to run** the following six lines now:

    mkdir tmp
    kmc -k21 -t8 -m20 -ci1 -cs2000000 -fa -sm /home/training/assembly/data/raw/ilThySylv1/ilThySylv1.filtered.1000.fasta.gz ilThySylv1.k21 tmp
    kmc_tools transform ilThySylv1.k21 histogram ilThySylv1.k21.hist -cx2000000
    awk '$2!=0' ilThySylv1.k21.hist > tmp/ilThySylv1.k21.hist
    mv tmp/ilThySylv1.k21.hist ilThySylv1.k21.hist

<ins>Obs</ins>: As running a KMC database requires a significant amount of time to finish, we have generated the final results for you. Now, you need to take the final histogram file and plot it with [GenomeScope](https://github.com/tbenavi1/genomescope2.0). You can use the [online browser version](http://qb.cshl.edu/genomescope/genomescope2.0/) or you can use the command lines bellow:


Change to the working folder:
    
    cd /home/training/assembly/workdir

Create the folder for GenomeScope results and run the following commands from it:

    mkdir genomescope 
    cd genomescope
    
For **idEriArbu1**, run: 

    singularity run  /home/training/assembly/images/genomescope-2.0.sif  -i /home/training/assembly/data/preqc/idEriArbu1/idEriArbu1.k21.hist  -o idEriArbu1.genomescope.output -k 21
    
or, for **ilThySylv1**, run:

    singularity run  /home/training/assembly/images/genomescope-2.0.sif  -i /home/training/assembly/data/preqc/ilThySylv1/ilThySylv1.k21.hist  -o ilThySylv1.genomescope.output -k 21

Change to the results folder using:

    cd /home/training/assembly/workdir/genomescope/idEriArbu1.genomescope.output
    
Or: 

    cd /home/training/assembly/workdir/genomescope/ilThySylv1.genomescope.output

#### **_Quiz_**

Now that you have (i) started a kmer counter run, (ii) plotted the pre-computed results in GenomeScope, you can analyse them and discuss them with your colleagues.  Answer the following questions:

&nbsp;&nbsp;&nbsp;&nbsp; 1. What is the estimated genome size?

&nbsp;&nbsp;&nbsp;&nbsp; 2. What is the estimated percentage of repeats?

&nbsp;&nbsp;&nbsp;&nbsp; 3. What is the estimated heterozygosity?

&nbsp;&nbsp;&nbsp;&nbsp; 4. What is the kmers coverage?

## Part 2 - Genome assembly and evaluation
In the morning you have generated and interpreted genome composition by analyzing the reads kmers. With that information in hand, now let’s assemble your reads. We will use the assembler [hifiasm](https://github.com/chhylp123/hifiasm). Have a try on the command below taking into consideration your species of choice:

The full data-sets for genome assembly can be downloaded using the links below. These datasets are large enough to demand a large amount of time for processing that's why in the next section we will try to run assembly on the subsets of 1000 reads already downloaded for you on the Virtual Machine.

http://darwin.cog.sanger.ac.uk/idEriArbu1.hifi.filtered.fasta.gz

http://darwin.cog.sanger.ac.uk/ilThySylv1.filtered.fasta.gz

### 2.1 - Genome assembly with hifiasm

Create a folder where your results will be stored and then run the assembler. 

For **idEriArbu1**, run:

    cd /home/training/assembly/workdir/
    mkdir idEriArbu1.hifiasm 
    cd idEriArbu1.hifiasm
    #now that you created your folder and changed to it, let’s run hifiasm
    singularity exec /home/training/assembly/images/hifiasm-0.15.3.sif hifiasm  -t8 --primary -o idEriArbu1 /home/training/assembly/data/raw/idEriArbu1/idEriArbu1.filtered.1000.fasta.gz

or, for **ilThySylv1**, run:

    cd /home/training/assembly/workdir/
    mkdir ilThySylv1.hifiasm 
    cd ilThySylv1.hifiasm
    singularity exec /home/training/assembly/images/hifiasm-0.15.3.sif hifiasm  -t8 --primary -o ilThySylv1 /home/training/assembly/data/raw/ilThySylv1/ilThySylv1.filtered.1000.fasta.gz
    
<ins>Obs</ins>: The results of hifiasm runs on the full datasets can be found in the `/home/training/assembly/data/assembly` folder. You can have a look at some general quantitative assembly statistics by running the following commands. 

For **idEriArbu1**, run:

    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.p_ctg.fa.gz
    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.a_ctg.fa.gz
    
or, for **ilThySylv1**, run:

    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.p_ctg.fa.gz
    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.a_ctg.fa.gz

### 2.2 - Running BUSCO

BUSCO is an important metric to evaluate the assembly completeness in gene level. 

To run BUSCO on **idEriArbu1**, run:

    cd /home/training/assembly/workdir/
    gunzip -c /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.p_ctg.fa.gz > /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.p_ctg.fa
    singularity exec /home/training/assembly/images/busco-5.0.0_cv1.sif busco -i /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.p_ctg.fa.gz -c 8 -o idEriArbu1.busco5 -m genome -l /home/training/assembly/data/busco/lineages/insecta_odb10 --offline

or, for **ilThySylv1**, run:
    
    cd /home/training/assembly/workdir/
    gunzip -c /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.p_ctg.fa.gz > /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.p_ctg.fa
    singularity exec /home/training/assembly/images/busco-5.0.0_cv1.sif busco -i /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.p_ctg.fa -c 8 -o ilThySylv1.busco5 -m genome -l /home/training/assembly/data/busco/lineages/insecta_odb10 --offline

We also have the BUSCO results for the hifiasm assembly. Download the precomputed results and have a look at the results, especially the "short summary" file.

For **idEriArbu1**, run:

    cd /home/training/assembly/workdir/
    wget https://darwin.cog.sanger.ac.uk/idEriArbu1.p_ctg.insecta_odb10.busco.tar.gz
    tar -xzvf idEriArbu1.p_ctg.insecta_odb10.busco.tar.gz
    cd idEriArbu1.p_ctg.insecta_odb10.busco

or, for **ilThySylv1**, run:
    
    cd /home/training/assembly/workdir/
    wget https://darwin.cog.sanger.ac.uk/ilThySylv1.p_ctg.insecta_odb10.busco.tar.gz
    tar -xzvf ilThySylv1.p_ctg.insecta_odb10.busco.tar.gz
    cd ilThySylv1.p_ctg.insecta_odb10.busco

#### **_Quiz_**

Looking at the statistics and BUSCO results for the hifiasm assembly, answer the following questions:

&nbsp;&nbsp;&nbsp;&nbsp; 5. Why do we have a “p” and “a” assembly results?

&nbsp;&nbsp;&nbsp;&nbsp; 6. What are the general statistics for the primary and alternate assemblies? How many contigs? N50? Total assembly length?

&nbsp;&nbsp;&nbsp;&nbsp; 7. How many complete BUSCO genes are identified in the primary contigs? How many of them were duplicated?

### 2.3 - Interpreting purge_dups outputs

As you learned in the theoretical classes, separation of haplotypes is usually not complete. We use a software called [purge_dups](https://github.com/dfguan/purge_dups) to further separate the haplotypes and to remove remaining haplotypic duplication from the primary assembly. Purge_dups has many steps and can take a significant amount of time to run. That's why the assembly produced by hifiasm was purged for you in advance and now you can generate the general statistics for the purged assemblies and evaluate the BUSCO results.

For **idEriArbu1**, run:

    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/idEriArbu1/purged/idEriArbu1.purged.fa.gz
    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/idEriArbu1/purged/idEriArbu1.purged.htigs.fa.gz

or, for **ilThySylv1**, run:

    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/ilThySylv1/purged/ilThySylv1.purged.fa.gz
    singularity exec /home/training/assembly/images/asmstats.sif asmstats /home/training/assembly/data/assembly/ilThySylv1/purged/ilThySylv1.purged.htigs.fa.gz

### 2.4 - Kmer plots to evaluate assembly quality

Apart from comparing quantitative statistics and the BUSCO score of the assemblies, there are other metrics and methods that help us evaluate assembly completeness and quality. One method discussed in the theoretical class is [merqury](https://github.com/marbl/merqury). Merqury evaluates assembly kmer completeness and duplication level by generating series of plots and statistics of a genome assembly vs. its reads counterparts. To do this comparison, you need to run merqury in all versions of your assembly, i.e. pre and post purging. The command-line bellow illustrates the way merqury can be run on a genome assembl, which in case of a large enough genome can also be a little time consumning. 

For **idEriArbu1**, run:

    # Run merqury for the raw assembly of idEriArbu1
    mkdir merqury.idEriArbu1.raw
    cd merqury.idEriArbu1.raw
    singularity exec /home/training/assembly/images/merqury-1.1.sif merqury.sh /home/training/assembly/data/preqc/idEriArbu1/idEriArbu1.10x.k21.meryl/ /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.p_ctg.fa.gz /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.a_ctg.fa.gz idEriArbu1 
    # Run merqury for the purged assembly of idEriArbu1
    cd /home/training/assembly/workdir/
    mkdir merqury.idEriArbu1.purged
    cd merqury.idEriArbu1.purged
    singularity exec /home/training/assembly/images/merqury-1.1.sif merqury.sh /home/training/assembly/data/preqc/idEriArbu1/idEriArbu1.10x.k21.meryl/ /home/training/assembly/data/assembly/idEriArbu1/purged/idEriArbu1.purged.fa.gz /home/training/assembly/data/assembly/idEriArbu1/purged/idEriArbu1.purged.htigs.fa.gz idEriArbu1 

or, for **ilThySylv1**, run:

    # Run merqury for the raw assembly of ilThySylv1:
    cd /home/training/assembly/workdir/
    mkdir merqury.ilThySylv1.raw
    cd merqury.ilThySylv1.raw
    singularity exec /home/training/assembly/images/merqury-1.1.sif merqury.sh /home/training/assembly/data/preqc/ilThySylv1/ilThySylv1.10x.k21.meryl/ /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.p_ctg.fa.gz /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.a_ctg.fa.gz ilThySylv1 
    # Run merqury for the purged assembly of ilThySylv1:
    cd /home/training/assembly/workdir/
    mkdir merqury.ilThySylv1.purged
    cd merqury.ilThySylv1.purged
    singularity exec /home/training/assembly/images/merqury-1.1.sif merqury.sh /home/training/assembly/data/preqc/ilThySylv1/ilThySylv1.10x.k21.meryl/ /home/training/assembly/data/assembly/ilThySylv1/purged/ilThySylv1.purged.fa.gz /home/training/assembly/data/assembly/ilThySylv1/purged/ilThySylv1.purged.htigs.fa.gz ilThySylv1  

As you put your command to run, please visit the final results folder to have a look at the merqury results for pre and post purging for your species. 

The finished output for **idEriArbu1** can be found here:

    /home/training/assembly/data/merqury/idEriArbu1
    /home/training/assembly/data/merqury/idEriArbu1_purged

or, for **ilThySylv1**: 

    /home/training/assembly/data/merqury/ilThySylv1
    /home/training/assembly/data/merqury/ilThySylv1_purged

### 2.5 - Evaluating pre and post genome purging

You have now tried to run and evaluate the results of several stages of the genome assembly process, which included running (i) hifiasm, (ii) purge_dups, (iii) merqury, (iv) asmstats and (v) BUSCO commands. 

Recapping, the assembly, purge and merqury results for each species are here. 

**idEriArbu1**:

    /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.p_ctg.fa.gz
    /home/training/assembly/data/assembly/idEriArbu1/idEriArbu1.a_ctg.fa.gz 
    /home/training/assembly/data/assembly/idEriArbu1/purged/idEriArbu1.purged.fa.gz
    /home/training/assembly/data/assembly/idEriArbu1/purged/idEriArbu1.purged.htigs.fa.gz

**ilThySylv1**:

    /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.p_ctg.fa.gz
    /home/training/assembly/data/assembly/ilThySylv1/ilThySylv1.a_ctg.fa.gz 
    /home/training/assembly/data/assembly/ilThySylv1/purged/ilThySylv1.purged.fa.gz
    /home/training/assembly/data/assembly/ilThySylv1/purged/ilThySylv1.purged.htigs.fa.gz

#### **_Quiz_**

Considering all the results generated before and after purging of your species answer the following questions:

&nbsp;&nbsp;&nbsp;&nbsp; 8. Considering the general statistics of the primary assembly before and after purging, what has happened with the assembly length?

&nbsp;&nbsp;&nbsp;&nbsp; 9. Considering the BUSCO results, what has changed from pre to post purge dups?

&nbsp;&nbsp;&nbsp;&nbsp; 10. Where in the merqury plots can you spot the difference in haplotypic retention pre and post purging?

## Part 3 - Scaffolding assembly with Hi-C data

A final part of the automated pipeline for a genome assembly is scaffolding. Here we will scaffold our Pacbio Hifi polished assembly with [Hi-C](https://pubmed.ncbi.nlm.nih.gov/22652625/) data using a software called [SALSA2](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1007273). Scaffolding procedure also involves many steps that you can read about in the Salsa manual and that demand significant amount of time. We will use our time to look at the general statistics and to interpret the contact maps that are generated after assemblies are scaffolded with Hi-C reads. Giving your species, download the results and unzip.

For **idEriArbu1**, run: 

    cd /home/training/assembly/workdir/
    wget https://darwin.cog.sanger.ac.uk/idEriArbu1.hifiasm.scaff.20210610.tar.gz
    tar -xzvf idEriArbu1.hifiasm.scaff.20210610.tar.gz
    cd idEriArbu1.hifiasm.scaff.20210610

or, for **ilThySylv1**, run:

    cd /home/training/assembly/workdir/
    wget https://darwin.cog.sanger.ac.uk/ilThySylv1.hifiasm.scaff.20210527.tar.gz
    tar -xzvf ilThySylv1.hifiasm.scaff.20210527.tar.gz
    cd ilThySylv1.hifiasm.scaff.20210527

have a look at the files inside:

- *hic - a HiC map file
- *agp - a file with coordinates of initial contigs with the scaffolds
- merqury
- busco5


The scaffold FASTA file is for **idEriArbu1** is:

    /home/training/assembly/data/assembly/idEriArbu1/scaff/idEriArbu1.scaffolds_FINAL.fasta

or, for **ilThySylv1**:

    /home/training/assembly/data/assembly/ilThySylv1/scaff/ilThySylv1.scaffolds_FINAL.fasta


<ins>Check the HiC contact map</ins>

Launch Juicebox using the command below:

    java -Xms512m -Xmx2048m -jar /home/training/assembly/bin/Juicebox_1.11.08.jar

and then open the .hic file by going to File > Open > Local ...

**idEriArbu1**:

    /home/training/assembly/workdir/idEriArbu1.hifiasm.scaff.20210610/salsa_scaffolds.hic
    
**ilThySylv1**
    
    /home/training/assembly/workdir/ilThySylv1.hifiasm.scaff.20210527/salsa_scaffolds.hic

#### **_Quiz_**

Checking the scaffolds and HiC contact map, answer the following questions:

&nbsp;&nbsp;&nbsp;&nbsp; 11. What is the final scaffold statistic for your assembly? Assembled size, N50, number of contigs, number of scaffolds...

&nbsp;&nbsp;&nbsp;&nbsp; 12. What is the final BUSCO results? Has the BUSCO changed since polishing?

&nbsp;&nbsp;&nbsp;&nbsp; 13. How the final merqury results look like?

&nbsp;&nbsp;&nbsp;&nbsp; 14. How does your final contact map looks like? What are the squares in the diagonal? What do the signals off diagonal represent?


## Part 4 - Quiz and presentation

We have now completed our analysis. Now, talk to your colleagues and make a few slides with the following information.

&nbsp;&nbsp;&nbsp;&nbsp; 15. What is your species name?

&nbsp;&nbsp;&nbsp;&nbsp; 16. What was the estimated genome size?

&nbsp;&nbsp;&nbsp;&nbsp; 17. Was is the primary assembled size before and after purging. Do the values correspond to the estimated genome size?

&nbsp;&nbsp;&nbsp;&nbsp; 18. How did BUSCO change after purging?

&nbsp;&nbsp;&nbsp;&nbsp; 19. How did statistics and BUSCO change after polishing?

&nbsp;&nbsp;&nbsp;&nbsp; 20. How did statistics change after scaffolding?

&nbsp;&nbsp;&nbsp;&nbsp; 21. In your HiC contact map, where are your main scaffolds represented?
