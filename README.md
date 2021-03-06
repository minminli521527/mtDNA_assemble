# mtDNA-assemble



* ## 1) canu assemble
- * ### 1.1) canu software installation
###### using conda
	$ conda create -n canu canu -y
	$ conda activate canu

- * ### 1.2) Assemble
- * #### 1.2.1) Correction
###### Parameter analysis：-p, specify the output prefix; -d specify the output result directory; genomeSize sets an estimated genome size, which is convenient for Canu to estimate the sequencing depth, the unit is g, m, k; maxThreads sets the maximum number of threads; minReadLength means only use the threshold value MinOverlapLength Set the minimum length of Overlap, increase minReadLength can increase the running speed, increase minOverlapLength can reduce the false positive overlap; In addition, you need to specify the type of input data, whether it is original sequencing data or processed (-pacbio-raw, Direct pacbio data obtained by direct sequencing; -pacbio-corrected corrected pacbio data; -nanopore-raw original nanopore data; -nanopore-corrected result corrected nanopore data); corOutCoverage: used to control how much data is used for error correction, for example, Arabidopsis thaliana is a 120M genome, and 12G data is obtained after 100X sequencing, if only the longest 6G data is to be used for error correction, then the parameter should be set to 50 (120m x 50), set a value greater than the sequencing depth , for example 120, means to use all data.
###### Corrected reads saved in 'mtDNA.correctedReads.fasta.gz'.
 	$ canu -correct -p mtDNA -d ./correct maxThreads=4 genomeSize=450k minReadLength=2000 minOverlapLength=500 corOutCoverage=120 corMinCoverage=2 -pacbio-raw ../data/mtDNA.fastq.gz
- * #### 1.2.2) Trim
###### Trimmed reads saved in 'mtDNA.trimmedReads.fasta.gz'.
  	$ canu -trim -p mtDNA -d ./trim maxThreads=8 genomeSize=450k minReadLength=2000 minOverlapLength=500 -pacbio-corrected ./correct/mtDNA.correctedReads.fasta.gz
- * #### 1.2.3) Assemble using canu
###### The error rate after error correction needs to be adjusted here. correctedErrorRate: the degree of tolerance of the difference between the overlapping parts of the two reads. Lowering this value can reduce the running time. If the coverage is high, it is recommended to reduce this value, it will affect utgOvlErrorRate. Multiple parameters can be tried in this step because of the speed comparison block.
###### error rate 0.035
  	$ canu -assemble -p mtDNA -d ./assemble_0.035 maxThreads=20  genomeSize=450k correctedErrorRate=0.035 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### Different canu versions have different commands. If an error is reported, you may try the following command, and the mtDNA.contigs.fasta under the final output file is the result file.
  	$ canu -assemble -p mtDNA -d ./assemble_0.035 maxThreads=20  genomeSize=450k errorRate=0.035 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### error rate 0.050, and the mtDNA.contigs.fasta under the final output file is the result file.
  	$ canu -assemble -p mtDNA -d ./assemble_0.050 maxThreads=20  genomeSize=450k correctedErrorRate=0.050 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq

* ## 2) Falcon assemble
- * ### 2.1) Falcon software installation
###### using conda
  	$  conda create -n pb-assembly pb-assembly
  	$  conda activate pb-assembly
- * ### 2.2) Prepare data
- * #### 2.2.1) Create input_fofn
###### The file input_fofn refers to the file containing the sequencing files name, each line must have the full path of the fasta file, and the file has been uploaded to this repository.
- * #### 2.2.2) Create the configuration file
###### The file fc_run.cfg has been uploaded to this repository.
###### And, it is best to download the template of the configuration file file fc_run.cfg and then modify it, otherwise it is easy to make mistakes. The configuration file controls the parameters used in various stages of Falcon assembly. However, at the beginning, we did not know which parameter is the optimal one. Adjustment. Of course, since there are already many species using Falcon for assembly, they can learn from their configuration files (https://pb-falcon.readthedocs.io/en/latest/parameters.html).
	$ wget https://pb-falcon.readthedocs.io/en/latest/_downloads/fc_run_ecoli_local.cfg
###### Most of the content of this file does not need to be modified, except for the following parameters. "input_fofn": the file input.fofn here is the file created in the previous step, it is recommended to put this file in the same directory of the configuration file, so that there is no need to change the path of the file in the configuration file. "genome_size", "seed_coverage", "length_cutoff", "length-cutoff_pr": these parameters control the amount of data used for error correction and the amount of data used for assembly, if you want the program to automatically determine the amount of data used for error correction when running, set "length_cutoff" to -1 ", set genome estimated size genome_size and depth seed_coverage for error correction at the same time. "jobqueue": here is a single host instead of a cluster, so in fact, just pick a name, but for SGE, you must choose the name of the queue that can be submitted. "xxx_concurrent_jobs": the number of jobs running at the same time is obviously more and faster, some configuration files have written 192, but for most people, there are not so many resources, blind writing more will only lead to server downtime.
- * ### 2.3) Assemble
###### Running Falcon is very simple, just prepare the configuration file and pass it to fc_run.py, and then let fc_run.py schedule all the required software to complete the genome assembly.
	$ fc_run.py fc_run_local.cfg
###### The final main result file generated is 2-asm-falcon/p_ctg.fa
###### 0-rawreads/: this directory stores the results of overlpping analysis and correction of raw subreads; 0-rawreads/cns-runs/cns_...*.fasta: stores the sequence information after correction; 1-preads_ovl/: this directory stores the overlaying of reads after correction The result; 2-asm-falcon/: this directory is the final result directory, the main result files are p_ctg.fa and a_ctg.fa.

* ## 3) ECAT2 assemble
###### Please refer to the URL for the process：http://blog.sciencenet.cn/blog-3406804-1203984.html.
- * ### 3.1) MECAT2 software installation
###### Download the binary version directly.
	$ wget https://github.com/xiaochuanle/MECAT2/releases/download/20192026/mecat2_20190226_linuax_amd64.tar.gz
	$ tar xzvf mecat2_20190226_linuax_amd64.tar.gz
###### Then add the main program to the environment variables, for example, my MECAT2 storage path is in /home/my/software/MECAT2/.
	$  export PATH=/home/my/software/MECAT2/Linux-amd64/bin:$PATH
###### View help
	$  mecat.pl
- * ### 3.2) Prepare data
- * #### 3.2.1) Data format
###### Currently, MECAT2 does not support gz compressed files, enter fastq or fasta.
- * #### 3.2.2) Configuration file：https://www.jianshu.com/p/176fc8105000
###### Using vim to modify the ecoli_config_file.txt file to config_file.txt.
	$ mecat.pl config ecoli_config_file.txt
- * ### 3.3) Assemble using MECAT2
###### Original data error correction
	$ mecat.pl correct config_file.txt
###### Assemble the corrected reads
	$ mecat.pl assemble config_file.txt
- * ### 3.4) Interpretation of results
###### Reads after error correction： 1-consensus/cns_reads.fasta.
###### Longest 30/20X error correction for trimming reads: 1-consensus/cns_final.fasta.
###### Trimmed reads: 2-trim_bases/trimReads.fasta.
###### Assembled contigs: 4-fsa/contigs.fasta.



* ## 4) NextDenovo assemble
* ### 4.1) NextDenovo software installation
###### Download the compiled binary version, which can be used directly without installation.
	$  wget https://github.com/Nextomics/NextDenovo/releases/download/v2.1-beta.0/NextDenovo.tgz
	$  tar -vxzf NextDenovo.tgz
###### Add executable permissions.
	$  cd NextDenovo
	$  chmod -R 755 *
###### Add to environment variables.
	$  export PATH=~/software/NextDenovo/:$PATH
	$  export PATH=~/software/NextDenovo/bin/:$PATH
###### If there is no problem at this time, you can use it directly.
###### The software needs python2.7 to run, it can be installed when the module is missing, but does not support python3.
	$  nextDenovo -h
- * ### 4.2) Prepare data
###### Record the actual location of the file in input.fofn, the file has been uploaded to this repository.
###### Copy and modify configuration files：https://www.jianshu.com/p/fa26792435eb  http://blog.sciencenet.cn/blog-3406804-1204832.html
	$ cp ~/software/NextDenovo/doc/run.cfg ./
	$ vi run.cfg
- * ### 4.3) Assemble using NextDenovo
###### For the final assembly sequence of NextDenovo, could see:03.ctg_graph/01.ctg_graph.sh.work/ctg_graph00/nextgraph.assembly.contig.fasta
	$ nextDenovo run.cfg



* ## 5) wtdbg2 assemble
* ### 5.1) wtdbg2 software installation
###### using conda
	$ conda create -n wtdbg
	$ conda install wtdbg -y
	$ conda activate wtdbg
	$ wtdbg2
* ### 5.2) Assemble using wtdbg2
###### Correction using canu
###### Corrected reads saved in 'mtDNA.correctedReads.fasta.gz'.
	$ conda activate canu
	$ canu -correct -p mtDNA -d ./correct maxThreads=4 genomeSize=450k minReadLength=2000 minOverlapLength=500 corOutCoverage=120 corMinCoverage=2 -pacbio-raw ../data/mtDNA.fastq.gz
###### Trim using canu
###### Trimmed reads saved in 'mtDNA.trimmedReads.fasta.gz'.
	$ canu -trim -p mtDNA -d ./trim maxThreads=8 genomeSize=450k minReadLength=2000 minOverlapLength=500 -pacbio-corrected ./correct/mtDNA.correctedReads.fasta.gz
###### Assemble using wtdbg2
###### Corrected sequence with canu.
	$ wtdbg2 -t 16 -i mtDNA.trimmedReads.fasta.gz -o prefix -L 5000
###### Get consensus sequence
	$ wtpoa-cns -t 2 -i prefix.ctg.lay.gz -o prefix.ctg.lay.fa
###### Polish and modify the genome sequence using three-generation sequencing reads. 
	$ minimap2 -t 2 -x map-pb -a prefix.ctg.lay.fa mtDNA.trimmedReads.fasta.gz | samtools view -Sb - > prefix.ctg.lay.map.bam
	$ samtools sort -o prefix.ctg.lay.map.sorted prefix.ctg.lay.map.bam
	$ samtools view prefix.ctg.lay.map.srt.bam | wtpoa-cns -t 2 -d prefix.ctg.lay.fa -i - -o prefix.ctg.lay.2nd.fa
###### The final assembly result is: prefix.ctg.lay.2nd.fa
	$ less prefix.ctg.lay.2nd.fa | grep ">"



* ## 6) Quickmerge the assembly results of different software
* ### 6.1) Quickmerge software installation
###### using conda
	$ conda create -n quickmerge
	$ conda install -c conda-forge -c bioconda quickmerge
	$ conda activatequickmerge
	$ merge_wrapper.py
* ### 6.2) Assemble using Quickmerge-method 1
###### One-step method: run a py script
###### The result is merged_out.fasta
###### Make pairwise comparisons many times to get the final result.
	$ merge_wrapper.py prefix.ctg.lay.2nd.fa  nextgraph.assembly.contig.fasta
* ### 6.3) Assemble using Quickmerge-method 2
###### Run in steps
######  Parameter analysis：-l|minmatch: set the minimum length of a single match (default 20). -p|prefix: set the prefix of the output file (default is out).
	$ nucmer -l 20 -p wtdbg2_nextDenovo prefix.ctg.lay.2nd.fa  nextgraph.assembly.contig.fasta
######  Parameter analysis：-i float: set the minimum alignment mark [0,100], the default is 0. -r: allows query overlaps (many to many). -q: allows reference overlaps (many to many)
	$ delta-filter -i 5 -r -q wtdbg2_nextDenovo.delta > wtdbg2_nextDenovo.rq.delta
###### Parameter analysis：In general, -l selects the N50 assembled from the reference (-r) sequence as the initial value, and calculates it using quast. -ml is generally greater than 5000. Sometimes it gives an error, try a few more times.
###### The result is merged_wtdbg2_nextDenovo.fasta
	$ quickmerge -d wtdbg2_nextDenovo.rq.delta -q ./prefix.ctg.lay.2nd.fa -r ./nextgraph.assembly.contig.fasta -hco 5.0 -c 1.5 -l 34171 -ml 6000 -p wtdbg2_nextDenovo



* ## 7) Preliminary mitochondrial sequence
###### Alignment database BLAST
* ### 7.1) BLAST software installation
###### using conda
	$ conda create blast blast -y
	$ conda activate blast
* ### 7.2) Blast
###### Reference: http://blog.sciencenet.cn/blog-3406804-1199850.html
###### A local version of the NCBI nucleic acid database (hereinafter referred to as NT library) is required. Through subsequent local BLAST, the contigs sequences that are aligned to the mitochondria are selected from the assembly results.
###### Perform BLAST alignment of all the contigs/scaffolds sequences and the nucleic acid sequences included in the NT library to locate the target sequence.
###### Nucleic acid comparison-blastn，specify the NT library path, and output the best hits for each sequence by default.
	$ blastn -db /database/nt/nt -query ./configs.fasta -out blast -num_threads 4 -num_descriptions 1 -num_alignments 1 -dust no
###### Transform the blast result format, get the link of the perl script: https://pan.baidu.com/s/1-HkUh_C9JgYH9q-J2R7ZDA
	$ perl blast_trans.pl spades_blast spades_blast.txt
###### According to the description of the annotation, extract the result of sequence alignment that hits "mitochondria".
	$ grep 'mitocho' blast.txt > blast.select.txt
###### View "blast.select.txt", only the results of known mitochondrial sequences that can be compared to the database are retained in this file, which can roughly determine which contigs are derived from mitochondria.



* ## 8) Contigs positioning and orientation
###### Need to combine manual process
###### After preliminary splicing, several assembly result fasta files were obtained. There are usually multiple contigs/scaffolds sequences in these fasta files (the whole sequence is only automatically assembled by software, which is almost impossible), the next step is to determine the relative position and orientation of these contigs/scaffolds sequences in the genome to continue to build up the complete circular mitochondrial genome sequence.
###### This step also needs to be done with the help of the reference genome. Align those assembled contigs / scaffolds sequences with the reference genome to determine the position and orientation relationship.
###### BLAST alignment results give the name of the reference mitochondrial genome sequence that best hits these contigs / scaffolds sequences. You can find one of the most similar reference genomes and download them in databases such as NCBI or EMBL by ID to help us determine the relative position and orientation (location and orientation) of these contigs / scaffolds sequences in the genome. In addition, the reference genome can also help us determine the final length range of our mitochondrial genome.
###### There are many tools that can achieve this function, visualization tools such as geneious, command-line tools such as MUMmer, and so on.
* ### 8.1) Collinear analysis
###### For example, use MUMmer for collinear analysis and positioning to determine the order of contigs / scaffolds
###### Reference genome sequence should be single species, not mixed species
	$ conda create mummer mummer -y
	$ conda activate mummer
	$ mkdir mummer && cd mummer
	$ nucmer --mum -p mitochondria ref_NC_030753.1.fasta mtDNA.contigs.fasta
	$ delta-filter -m mitochondria.delta > mitochondria.filter
	$ show-coords -T -r -l mitochondria.filter > mitochondria.1coords
	$ mummerplot --postscript -p mitochondria mitochondria.delta
	$ ps2pdf mitochondria.ps mitochondria.pdf
###### For the results of collinearity analysis, could directly view the content in the text result file "mitochondria.1coords" and record the details of collinearity matching of the assembled scaffolds sequence and the reference genome sequence. Or more intuitively, view the resulting collinearity result graph, where purple/red indicates forward and blue indicates reverse.
* ### 8.2) Extract sequences in sequence
###### combine tig_2.fasta in order.
	$ samtools faidx mtDNA.contigs.fasta
	$ samtools faidx mtDNA.contigs.fasta tig00000011 > tig00000011.fa
	$ ......
###### Extract reverse complementary sequence.
	$ seqkit seq -t dna tig00000045.fa -r -p> tig00000045-RC.fa
###### Combining different sequences.
	$ cat tig000000011.fa tig00000045-RC.fa tig00000044.fa tig00000009-RC.fa tig00000027-RC.fa tig00000030-RC.fa tig00000058-RC.fa tig00000032.fa tig00000029.fa tig00000046.fa tig00000018.fa tig00000068.fa tig00000040.fa tig00000050.fa tig00000062-RC.fa tig00000044-RC.fa tig00000024-RC.fa tig00000020.fa tig00000054.fa tig00000061-RC.fa tig00000019.fa tig00000009.fa tig00000044-RC.fa tig00000011.fa tig00000060.fa tig00000038-RC.fa tig00000021-RC.fa tig00000013-RC.fa tig00000043-RC.fa tig00000012.fa tig00000046-RC.fa tig00000035-RC.fa tig00000014-RC.fa > tig_2.fasta
###### Repeat step 1 for tig_2.fasta and perform collinearity analysis again.



* ## 9) Genomic hole filling and scaffold connection
###### Using PBJelly2, and Pacbio reads
###### If the previous step did not completely loop the mitochondrial genome and there is a gap in the middle, then this part of the content will be useful.
* ### 9.1) PBJelly2 software installation
###### Follow the steps to install PBJelly2: https://sr-c.github.io/2019/07/02/PBJelly-and-blasr-installation/ ，http://cache.baiducontent.com/c?m=9f65cb4a8c8507ed4fece763105392230e54f73266808c4b2487cf1cd4735b36163bbca63023644280906b6677ed1a0dbaab6b66725e60e1948ad8128ae5cc6338895734&p=c363c64ad4d914f306bd9b78084d&newp=8f73c64ad48811a05ee8c6365f4492695d0fc20e38d3d701298ffe0cc4241a1a1a3aecbf2d211301d7c47f6006a54359e9fb30703d0034f1f689df08d2ecce7e64&user=baidu&fm=sc&query=pbjelly2&qid=e4e870de000cfaa0&p1=9 .
###### Special attention: need to run in python2.7 environment, otherwise report error.py, ## line
###### Special attention: conda install networkx == 1.11
* ### 9.2) Run
###### First create the configuration file Protocol.xml.
###### Then run the next 6 steps in sequence:
	$ Jelly.py setup Protocol.xml
	$ Jelly.py mapping Protocol.xml
	$ Jelly.py support Protocol.xml
	$ Jelly.py extraction Protocol.xml
	$ Jelly.py assembly Protocol.xml -x "--nproc=24"
	$ Jelly.py output Protocol.xml
###### The --nproc parameter sets the number of running threads.
###### The output file is jelly.out.fasta.
###### For scaffold connection, using PBJelly2.
	$ grep -Ho N jelly.out.fasta | uniq -c
