RiHetCaller Version 1.0 for linux only

Author Gonghao <mygonghao@163.com>
School of life science, Huizhou University, Guangdong Province, China


All the C excutables are in the current directory and the souce can be obtained upon resquest.
We compile the programme with g++ (version 4.8.2) and X86-64 redhat linux system. You can
run it directly. 

Note that some of our excutable will require the samtools pacakges to compile. As we use the 
samtools API and Beagle in our programme, so you should have the related pacakges installed.
the samtools can be downloaded from the following link
http://samtools.sourceforge.net/
the Beagle software pacakge can be downloaded from the following link.
https://faculty.washington.edu/browning/beagle/beagle.html

The common work flow of for this pipleline should be as follows:

First, Seqeunce reads are algined to the reference genome by the alignment software to generate bam
file with header added. there are some excellent alignment softwares avaiable now, such as BWA,Bowtie,
Bowtie2 and so on.

Second, Our pipeline can be used to call snp from the bam file to generate the final snp dataset.
As we carry out the analysis in a step by step process you can also use your own cretria to filter
the snp dataset and generate the final snp dataset. Then we can compute the genotype likelihood
for the snp dataset and it can used directly as input file for beagle ( an excellent genotype 
imputation software). If missing snp didn't affect your analysis the following process can ingored
and you can use this dataset for your analysis. 

Third, Beagle software is used to impute the missing genotype from the dataset. Then we get final
beagle imputed snpd dataset.

One example for the run of the workflow is listed below. for the details about the output file format and
the column header information you can check the "./example_data/README.txt" for details. One example work
flow with example dataset is also explained there.
--------------------------------------------------------------------------------------------------------------
Short reads alignment 
Alignment software is used to align the sequence reads to reference genome, we take BWA as an example.
for pair end reds. the command can be as follows:
"./bwa mem -M -t 4 <reference file> <left end reads> < right end reads> | samtools view -bS -| 
samtools sort - <outfile name>". 

Take care that the reference genome should be indexed with related software.It can generate the sorted bam file.

--------------------------------------------------------------------------------------------------------------
Call snp from bam file

"RiHetCaller_step1" program can be used to find the snp from the bam file. it will find the mismatch base pair in the aligned
software to generate primary snp dataset. The example command is as follows:
"./RiHetCaller_step1 <bam file> <reference file> <output file>".

the output file is not sorted and it didn't affect the downstream analysis.
--------------------------------------------------------------------------------------------------------------
Summarize snp to generate candidate polymorphic site.

"./RiHetCaller_step2" program can be used to summarize the snp dataset generated by the program of "Call_snp". the
input file is the list of file generate by "Call_snp". it will output
SNP count for each mismatch with a total count bigger than a threhold in the whole population. the example command is as follows:

"./RiHetCaller_step2 <reference file> <Pile filelist> <output file> <min snp number>

you can filter ouput file with your own cretria two and use your filtered file for the downstream analysis.
--------------------------------------------------------------------------------------------------------------

Pile up all the base pair on the chosen polymorphic site.
"./RiHetCaller_step3" program can pile all the base pair that is aligned to the chosen snp dataset from the output file 
of "sum_snp". It will output both the algined base pair for the chosen site and it can also ouput the related quality socore.
the example command is as follows:

"./RiHetCaller_step3 <bam file> < reference file> <chosen poly site> <output file>

we denote missing as "*" in the output file. We denote the output file as mpile format to differe from
the file generated by "Cal_snp".

--------------------------------------------------------------------------------------------------------------
Filter Snp on missing rate and minor allele frequency
"./RiHetCaller_step4" will summarize all the base pair allele frequency on the chosen snp dataset generated by the "Sum_snp".
the input file is the mpile fle generated by "Choose_pile". You need to specify the missing rate and minor allele frequency
for filtering of the single nucleotide polymorphism site and it will generate the final chosen polymorphic sites.

the example command is as follows:
"./RiHetCaller_step4 <mpile file list> <Minor allele frequency > < missing rate> <fileout>

you also filter the chosen site by your self customized method and use it for downstream analysis.
--------------------------------------------------------------------------------------------------------------

Calculate genotype Likelihood for imputation.

"./RiHetCaller_step5" will generate genotype likelihood for each accession on the chosen polymorphic site using a bimodial model.
the input file is the mpile file generated by "Choose_pile" and the final snp site generated by "Filter_snp". We incoporate
the popualtion allele frequency on each polymorphic site in the genotype likelihod calculation. the example
command is as follows:
"./RiHetCaller_step5 <mpile filelist> <chosen poly site> <output file>

the output file is the VCF 4.0 format and it can be used directly as input file for beagle imputation.

--------------------------------------------------------------------------------------------------------------

Beagle missing genotype imputation

You can use beagle4.0 or any other genotype imputation software that accepts VCF 4.0 as the input format.
the input file is output file generated by the "Cal_prob" program. one example command is as follows:

"java -jar beagle.jar gl=<input file> out=<output file>".

--------------------------------------------------------------------------------------------------------------
License
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the 'Software'), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
