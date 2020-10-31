Welcome to use COCACOLA (binning metagenomic contigs using sequence COmposition, read CoverAge, CO-alignment, and paired-end read LinkAge)!
===================

COCACOLA is a general framework that combines different types of information: sequence COmposition, CoverAge across multiple samples, CO-alignment to reference genomes and paired-end reads LinkAge to automatically bin contigs into OTUs. Furthermore, COCACOLA seamlessly embraces customized prior knowledge to facilitate binning accuracy.

News: Python version of COCACOLA is available now! 


Matlab-version of COCACOLA
============
----------
Description
---------------
This package contains the following files and directories.

	blocknnls.m -> non-negative least square parallel wrapper
	calCorrMat.m -> calculate the pairwise correlation of feature-object matrix
	calInternalIdx.m -> calculate TSS minimization index
	clustAgg_Lmethod.m -> eliminate suspicious clusters using bottom-up L Method
	clustAgg_SepCond.m -> merge closely mixed clusters by separable conductance
	example.m -> a demo on simulated 'strain' dataset
	myKmeansPar.m -> implementation of k-means clustering
	myNMF.m -> key algorithm
	data -> example datasets directory
	nmf_bpas -> non-negative least square algorithm developed by Kim and Park [1,2]
	vlfeat-0.9.20.zip -> open source library implements very fast version of k-means [3]

Please try to execute 'example.m' to learn how to use this software given the input generated by CONCOCT [4]. And please check the description of 'myNMF.m' for the detailed usage of the algorithm.


----------
Setup vlfeat
---------------

Before using COCACOLA, the users are expected to setup the enviroment of vlfeat.

First of all, we change the working directory to the current folder and unzip the vlfeat.
```sh
$ unzip vlfeat-0.9.20.zip
```
Next, we open the matlab and change the working directory to the current folder
```matlab
>> cd vlfeat-0.9.20/toolbox
>> vl_setup
>> cd ..
>> cd ..
```

----------
Preprocessing
---------------

The preprocessing steps aim to extract coverage profile and sequence composition profile as input to our program, which can be tackled by CONCOCT [4]. Here we provide a step-by-step guidance on simulated 'strain' dataset: 

First of all, we changes the current working directory to the data folder.
```sh
$ cd data/StrainMock/
```
We need to set the directories of dependency software in environmental variables. Notice that the setting in the example may differ from your own.
```sh
$ CONCOCT_dir=/home/cmb-panasas2/ylu465/CONCOCT-master
$ BOWTIE_dir=/home/cmb-panasas2/ylu465/bowtie2-2.2.3
$ SAMTOOLS_dir=/home/cmb-panasas2/ylu465/samtools-1.1/bin
```
####<i class="icon-cog"> Map reads to assembly contigs

First create the index on the assembly contigs using [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml):
```sh
$ cd contigs/
$ $BOWTIE_dir/./bowtie2-build -f StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta
$ cd ..
```
Next, we map the reads to assembly contigs for each samples, here we take Sample1006 as example. We can either integrated solution by CONCOCT [4] 
```sh
$ export MRKDUP=/home/cmb-panasas2/ylu465/picard-tools-1.77/MarkDuplicates.jar
$ bash $CONCOCT_dir/scripts/map-bowtie2-markduplicates.sh -ct 10 -p '-f' samples/Sample1006/Sample1006_1.fasta samples/Sample1006/Sample1006_2.fasta pair contigs/StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta Sample1006 samples/Sample1006/
```
or we can equivalently use [SAMTools](http://samtools.sourceforge.net/):
```sh
$ $BOWTIE_dir/./bowtie2 -f --fr -x contigs/StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta -1 samples/Sample1006/Sample1006_1.fasta -2 samples/Sample1006/Sample1006_2.fasta -S samples/Sample1006/Sample1006_pair.sam -p 10
$ $SAMTOOLS_dir/./samtools view -b -S samples/Sample1006/Sample1006_pair.sam -o samples/Sample1006/Sample1006_pair.bam
$ $SAMTOOLS_dir/./samtools sort -T samples/Sample1006/ -o samples/Sample1006/Sample1006_pair-smds.bam samples/Sample1006/Sample1006_pair.bam
$ $SAMTOOLS_dir/./samtools index samples/Sample1006/ -o samples/Sample1006/Sample1006_pair-smds.bam
$ rm samples/Sample1006/Sample1006_pair.sam
$ rm samples/Sample1006/Sample1006_pair.bam
```


####<i class="icon-cog"> Generate coverage table

Create a table with the coverage of each contig per sample using the bam files. 
```sh
$ python $CONCOCT_dir/scripts/gen_input_table.py contigs/StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta samples/Sample1006/Sample1006_pair-smds.bam samples/Sample1023/Sample1023_pair-smds.bam samples/Sample118/Sample118_pair-smds.bam samples/Sample120/Sample120_pair-smds.bam samples/Sample127/Sample127_pair-smds.bam samples/Sample134/Sample134_pair-smds.bam samples/Sample177/Sample177_pair-smds.bam samples/Sample215/Sample215_pair-smds.bam samples/Sample230/Sample230_pair-smds.bam samples/Sample234/Sample234_pair-smds.bam samples/Sample244/Sample244_pair-smds.bam samples/Sample261/Sample261_pair-smds.bam samples/Sample263/Sample263_pair-smds.bam samples/Sample290/Sample290_pair-smds.bam samples/Sample302/Sample302_pair-smds.bam samples/Sample321/Sample321_pair-smds.bam samples/Sample330/Sample330_pair-smds.bam samples/Sample343/Sample343_pair-smds.bam samples/Sample353/Sample353_pair-smds.bam samples/Sample371/Sample371_pair-smds.bam samples/Sample387/Sample387_pair-smds.bam samples/Sample409/Sample409_pair-smds.bam samples/Sample416/Sample416_pair-smds.bam samples/Sample424/Sample424_pair-smds.bam samples/Sample427/Sample427_pair-smds.bam samples/Sample454/Sample454_pair-smds.bam samples/Sample477/Sample477_pair-smds.bam samples/Sample482/Sample482_pair-smds.bam samples/Sample491/Sample491_pair-smds.bam samples/Sample495/Sample495_pair-smds.bam samples/Sample507/Sample507_pair-smds.bam samples/Sample509/Sample509_pair-smds.bam samples/Sample512/Sample512_pair-smds.bam samples/Sample522/Sample522_pair-smds.bam samples/Sample548/Sample548_pair-smds.bam samples/Sample564/Sample564_pair-smds.bam samples/Sample609/Sample609_pair-smds.bam samples/Sample616/Sample616_pair-smds.bam samples/Sample620/Sample620_pair-smds.bam samples/Sample624/Sample624_pair-smds.bam samples/Sample631/Sample631_pair-smds.bam samples/Sample687/Sample687_pair-smds.bam samples/Sample710/Sample710_pair-smds.bam samples/Sample712/Sample712_pair-smds.bam samples/Sample717/Sample717_pair-smds.bam samples/Sample733/Sample733_pair-smds.bam samples/Sample746/Sample746_pair-smds.bam samples/Sample759/Sample759_pair-smds.bam samples/Sample767/Sample767_pair-smds.bam samples/Sample803/Sample803_pair-smds.bam samples/Sample812/Sample812_pair-smds.bam samples/Sample827/Sample827_pair-smds.bam samples/Sample838/Sample838_pair-smds.bam samples/Sample853/Sample853_pair-smds.bam samples/Sample868/Sample868_pair-smds.bam samples/Sample871/Sample871_pair-smds.bam samples/Sample872/Sample872_pair-smds.bam samples/Sample882/Sample882_pair-smds.bam samples/Sample904/Sample904_pair-smds.bam samples/Sample906/Sample906_pair-smds.bam samples/Sample919/Sample919_pair-smds.bam samples/Sample943/Sample943_pair-smds.bam samples/Sample961/Sample961_pair-smds.bam samples/Sample983/Sample983_pair-smds.bam > input/cov_inputtableR.tsv
```


####<i class="icon-cog"> Generate composition table
```sh
$ python $CONCOCT_dir/scripts/fasta_to_features.py contigs/StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta 9417 4 input/kmer_4_tmp.csv
```
**Notice:** Here 9417 is the number of contigs number in contigs/StrainMock_Contigs_cutup_10K_nodup_filter_1K.fasta



Python-version of COCACOLA
============
----------
Setup Anaconda
---------------
Here we describe using Anaconda to run python version of COCACOLA on Linux/Unix. Anaconda is a tool to isolate your python installation, which allows you to have multiple parallel installations using different versions of different packages, and gives you a very convenient and fast way to install the most common scientific python packages. Anaconda can be downloaded from [here](https://www.continuum.io/downloads)

After installing Anaconda, create a new environment that will contain the COCACOLA installation:

```sh
$ conda create -n cocacola_env python=2.7.6
```

After creating the Anaconda environment, run the following command to activate it:

```sh
$ source activate cocacola_env
```

After that,  install the COCACOLA dependencies into this environment:

```sh
$ conda install numpy scipy pandas scikit-learn cvxopt
```

----------
Download
---------------

-> 1. Download the python version of COCACOLA from [here](https://www.dropbox.com/s/ciebt2y5h7pb9r2/COCACOLA-python.zip?dl=0)
-> 2. Unzip it
-> 3. Change the working directory to the unzipped folder
-> 4. Notice that if you are not using Linux/Unix, you need to re-compile the FragGeneScan and hmmer under the auxiliary folder, which is used to estimate the initial bin number by single-copy genes (credits owe to MaxBin2.0 [5]).

----------
Usage
---------------

The command can be viewed by typing ``python cocacola.py -h`` on the command line:

> - Uage: cocacola.py [-h] [--contig_file CONTIG_FILE]
                   [--abundance_profiles ABUNDANCE_PROFILES]
                   [--composition_profiles COMPOSITION_PROFILES]
                   [--edge_list EDGE_LIST] [--output OUTPUT]
                   [--clusters CLUSTERS]

> - optional arguments:-

	-h, --help: show this help message and exit. 
				
	--contig_file CONTIG_FILE: 
    					The contigs file.
	
	--abundance_profiles ABUNDANCE_PROFILES: 
    					The abundance profiles, containing a table where each
                        row correspond to a contig, and each column correspond
                        to a sample. All values are separated with tabs.
    
    --composition_profiles: 
    					The composition profiles, containing a table where
                        each row correspond to a contig, and each column
                        correspond to the kmer composition of particular kmer.
                        All values are separated with comma.
				
	--edge_list EDGE_LIST: 
    					The edges encoding either the co-alignment or the
                        pair-end linkage information, one row for one edge in
                        the format: contig_name_A contig_name_B weight. The
                        edge is undirected.
	
	--output OUTPUT: 
    					The output file, storing the binning result. If not
                        specified, the result is displayed directly on the
                        console.
    
    --clusters CLUSTERS: 
    					Specify the number of clusters. If not specified, the
                        cluster number is estimated by single-copy genes.

----------
Example without Using Additional Information
---------------

```sh
$ python cocacola.py --contig_file data/SpeciesMock/input/SpeciesMock_Contigs_cutup_10K_nodup_filter_1K.fasta --abundance_profiles data/SpeciesMock/input/cov_inputtableR.tsv --composition_profiles data/SpeciesMock/input/kmer_4.csv --output data/SpeciesMock/result.csv
```
----------
Example Using Additional Information
---------------

```sh
$ python cocacola.py --contig_file data/SpeciesMock/input/SpeciesMock_Contigs_cutup_10K_nodup_filter_1K.fasta --abundance_profiles data/SpeciesMock/input/cov_inputtableR.tsv --composition_profiles data/SpeciesMock/input/kmer_4.csv --edge_list data/SpeciesMock/input/linkage.txt --output data/SpeciesMock/result_link.csv
```
```sh
$ python cocacola.py --contig_file data/SpeciesMock/input/SpeciesMock_Contigs_cutup_10K_nodup_filter_1K.fasta --abundance_profiles data/SpeciesMock/input/cov_inputtableR.tsv --composition_profiles data/SpeciesMock/input/kmer_4.csv --edge_list data/SpeciesMock/input/coalign.txt --output data/SpeciesMock/result_coalign.csv
```


----------
Contacts and bug reports
------------------------
Please send bug reports, comments, or questions to 

Yang Lu: [ylu465@usc.edu](mailto:ylu465@usc.edu)

Prof. Fengzhu Sun: [fsun@usc.edu](mailto:fsun@usc.edu)


----------
Copyright and License Information
---------------------------------
Copyright (C) 2016 University of Southern California, Yang Lu

Authors: Yang Lu

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program. If not, see http://www.gnu.org/licenses/.


----------
References
---------------------------------
[1] Kim, H., Park, H.: Nonnegative matrix factorization based on alternating nonnegativity constrained least squares and active set method. SIAM Journal on Matrix Analysis and Applications 30(2), 713-730 (2008)

[2] Kim, J., He, Y., Park, H.: Algorithms for nonnegative matrix and tensor factorizations: A unified view based on block coordinate descent framework. Journal of Global Optimization 58(2), 285-319 (2014)

[3] http://www.vlfeat.org/

[4] Alneberg, J., Bjarnason, B.S., de Bruijn, I., Schirmer, M., Quick, J., Ijaz, U.Z., Lahti, L., Loman, N.J., Andersson, A.F., Quince, C.: Binning metagenomic contigs by coverage and composition. Nature Methods 11(11), 1144-1146 (2014)

[5] Wu, Yu-Wei, Blake A. Simmons, and Steven W. Singer. "MaxBin 2.0: an automated binning algorithm to recover genomes from multiple metagenomic datasets." Bioinformatics, btv638 (2015)

Last update: 05-Mar-2017
