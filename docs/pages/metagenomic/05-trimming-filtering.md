---
title: "Trimming and Filtering"
teaching: 30
exercises: 25
questions:
- "How can we get rid of sequence data that does not meet our quality standards?"
objectives:
- "Clean FASTQ reads using Trimmomatic."
- "Select and set multiple options for command line bioinformatic tools."
- "Write `for` loops with two variables."
keypoints:
- "The options you set for the command-line tools you use are important!"
- "Data cleaning is essential at the beginning of metagenomics workflows."
- "Use Trimmomatic to get rid of adapters and low-quality bases or reads."
- "Carefully fill in the parameters and options required to call a function in the bash shell."
- "Automate repetitive workflows using for loops."
---

!!! info "Lesson overview"
    **Teaching:** 15 min  
    **Exercises:** 15 min  

    **Questions**
    - How can we get rid of sequence data that does not meet our quality standards?

    **Objectives**
    - Clean FASTQ reads using Trimmomatic.

## Cleaning reads

In the last episode, we took a high-level look at the quality
of each of our samples using `FastQC`. We visualized per-base quality
graphs showing the distribution of the quality at each base across
all the reads from our sample. This information helps us to determine 
the quality threshold we will accept, and thus, we saw information about
which samples fail which quality checks. Some of our samples failed 
quite a few quality metrics used by FastQC. However, this does not mean
that our samples should be thrown out! It is common to have some 
quality metrics fail, which may or may not be a problem for your 
downstream application. For our workflow, we will remove some low-quality sequences to reduce our false-positive rate due to 
sequencing errors.

To accomplish this, we will use a program called
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). This 
useful tool filters poor quality reads and trims poor quality bases 
from the specified samples.

## Trimmomatic options

Trimmomatic has a variety of options to accomplish its task. 
If we run the following command, we can see some of its options:

~~~
$ trimmomatic
~~~
{: .bash}

Which will give you the following output:
~~~
Usage: 
       PE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] [-validatePairs] [-basein <inputBase> | <inputFile1> <inputFile2>] [-baseout <outputBase> | <outputFile1P> <outputFile1U> <outputFile2P> <outputFile2U>] <trimmer1>...
   or: 
       SE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] <inputFile> <outputFile> <trimmer1>...
   or: 
       -version
~~~
{: .output}

This output shows that we must first specify whether we have paired-end (`PE`) or single-end (`SE`) reads. Next, we will specify with which flags we 
want to run Trimmomatic. For example, you can specify `threads` 
to indicate the number of processors on your computer that you want Trimmomatic 
to use. In most cases, using multiple threads(processors) can help to run the 
trimming faster. These flags are unnecessary, but they can give you more control
over the command. The flags are followed by **positional arguments**, meaning 
the order in which you specify them is essential. In paired-end mode, Trimmomatic 
expects the two input files and then the names of the output files. These files are 
described below. While in single-end mode, Trimmomatic will expect one file 
as input, after which you can enter the optional settings and, lastly, the 
name of the output file.

| Option    | Meaning |  
| ------- | ---------- |  
|  \<inputFile1>  | input forward reads to be trimmed. Typically the file name will contain an `_1` or `_R1` in the name.|  
| \<inputFile2> | Input reverse reads to be trimmed. Typically the file name will contain an `_2` or `_R2` in the name.|  
|  \<outputFile1P> | Output file that contains surviving pairs from the `_1` file. |  
|  \<outputFile1U> | Output file that contains orphaned reads from the `_1` file. |  
|  \<outputFile2P> | Output file that contains surviving pairs from the `_2` file.|  
|  \<outputFile2U> | Output file that contains orphaned reads from the `_2` file.|  
  
The last thing Trimmomatic expects to see is the trimming parameters:

| step   | meaning |  
| ------- | ---------- |  
| `ILLUMINACLIP` | Perform adapter removal. |  
| `SLIDINGWINDOW` | Perform sliding window trimming, cutting once the average quality within the window falls below a threshold. |  
| `LEADING`  | Cut bases off the start of a read if below a threshold quality.  |  
|  `TRAILING` |  Cut bases off the end of a read if below a threshold quality. |  
| `CROP`  |  Cut the read to a specified length. |  
|  `HEADCROP` |  Cut the specified number of bases from the start of the read. |  
| `MINLEN`  |  Drop an entire read if it is below a specified length. |  
|  `TOPHRED33` | Convert quality scores to Phred-33.  |  
|  `TOPHRED64` |  Convert quality scores to Phred-64. |  

Understanding the steps you are using to
clean your data is essential. We will use only a few options and trimming steps in our
analysis. For more information about the Trimmomatic arguments
and options, see [the Trimmomatic manual](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf).

<a href="../fig/03-03-01.png">
  <img src="../fig/03-03-01.png" alt="Diagram showing the parts of the sequence that are reviewed by each parameter and the parts that are maintained or discarded at the end of the process. The Illuminaclip parameter removes the adapters, and the SlidingWindow scans the read by sections and removes a part of the read below the quality threshold. We remain with a trimmed read that has a valid quality." />
</a>

However, a complete command for Trimmomatic will look something like the command below. This command is an example and will not work, as we do not have the files it refers to:

~~~
$ trimmomatic PE -threads 4 SRR_1056_1.fastq SRR_1056_2.fastq \
              SRR_1056_1.trimmed.fastq SRR_1056_1un.trimmed.fastq \
              SRR_1056_2.trimmed.fastq SRR_1056_2un.trimmed.fastq \
              ILLUMINACLIP:SRR_adapters.fa SLIDINGWINDOW:4:20
~~~
{: .bash}

In this example, we have told Trimmomatic:

| code   | meaning |
| ------- | ---------- |
| `PE` | that it will be taking a paired-end file as input |
| `-threads 4` | to use four computing threads to run (this will speed up our run) |
| `SRR_1056_1.fastq` | the first input file name. Forward |
| `SRR_1056_2.fastq` | the second input file name. Reverse |
| `SRR_1056_1.trimmed.fastq` | the output file for surviving pairs from the `_1` file |
| `SRR_1056_1un.trimmed.fastq` | the output file for orphaned reads from the `_1` file |
| `SRR_1056_2.trimmed.fastq` | the output file for surviving pairs from the `_2` file |
| `SRR_1056_2un.trimmed.fastq` | the output file for orphaned reads from the `_2` file |
| `ILLUMINACLIP:SRR_adapters.fa`| to clip the Illumina adapters from the input file using the adapter sequences listed in `SRR_adapters.fa` |
|`SLIDINGWINDOW:4:20` | to use a sliding window of size 4 that will remove bases if their Phred score is below 20 |

 
 
## Running Trimmomatic on Galaxy

Now, we will run Trimmomatic on our data.
Instead of using a command, we are going to use Galaxy again !


We are going to run Trimmomatic on one of our paired-end samples.
First, we will do it with JP4D.

### Create a new history 
First, let's create a new history, and name it "Trimming and Filtering"
(Ajouter une image de creation d'history).
Then using the History Multiview, we can move the fastq from the previous analysis (FastQC). (JP4D collection)
Then, we can type "Trimmomatic" inside the "Tools" .  


While using FastQC, we saw that Universal adapters were present 
in our samples. The adapter sequences came with the installation of 
Trimmomatic and it is located in our current directory in the database  `TruSeq3`.

We will also use a sliding window of size 4 that will remove bases if their
Phred score is below 20 (like in our example above). We will also
discard any reads that do not have at least 25 bases remaining after
this trimming step.
(Put a picture of the parameters to use inside Trimmomatic)

Once you have selected all the good parameters, you can click on "Run".
This command will take a few minutes to run.
Open the output " Trimmomatic on collection X (log file)

~~~
TrimmomaticPE: Started with arguments:
 JP4D_R1.fastq.gz JP4D_R2.fastq.gz JP4D_R1.trim.fastq.gz JP4D_R1un.trim.fastq.gz JP4D_R2.trim.fastq.gz JP4D_R2un.trim.fastq.gz SLIDINGWINDOW:4:20 MINLEN:35 ILLUMINACLIP:TruSeq3-PE.fa:2:40:15
Multiple cores found: Using 2 threads
Using PrefixPair: 'TACACTCTTTCCCTACACGACGCTCTTCCGATCT' and 'GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT'
ILLUMINACLIP: Using 1 prefix pairs, 0 forward/reverse sequences, 0 forward only sequences, 0 reverse only sequences
Quality encoding detected as phred33
Input Read Pairs: 1123987 Both Surviving: 751427 (66.85%) Forward Only Surviving: 341434 (30.38%) Reverse Only Surviving: 11303 (1.01%) Dropped: 19823 (1.76%)
TrimmomaticPE: Completed successfully
~~~
{: .output}

> ## Exercise 1: What did Trimmomatic do?
>
> Use the output from your Trimmomatic command to answer the
> following questions.
>
> 1) What percent of reads did we discard from our sample?  
> 2) What percent of reads did we keep both pairs?
>
>> ## Solution
>> 1) 1.76%    
>> 2) 66.85%
> {: .solution}
{: .challenge}

You may have noticed that Trimmomatic automatically detected the
quality encoding of our sample (phred33). It is always a good idea to
double-check this or manually enter the quality encoding.

## Exercise 2 
The output files are also FASTQ files. 
!!! question "Should the output fastqfile file be smaller or bigger than the input file"
        ??? success "It should be smaller than our input file because we have removed reads."


We have just successfully run Trimmomatic on one of our FASTQ files!
Now we need to do it on our other sample : JC1A.


We have completed the trimming and filtering steps of our quality
control process! 

> ## Bonus Exercise: Quality test after trimming
>
> Now that our samples have gone through quality control, they should perform
> better on the quality tests run by FastQC. 

 Rerun the FastQC Analysis to check if our sequences have been succesfully filtered and trimmed.
 
  After trimming and filtering, our overall quality is much higher, 
  we have a distribution of sequence lengths, and more samples pass 
  adapter content. However, quality trimming is not perfect, and some
  programs are better at removing some sequences than others. Trimmomatic 
  did pretty well, though, and its performance is good enough for our workflow.


Now that the Quality Control Process step is done, we can move on to the **Taxoxomic assignement of our samples**.   


!!! Success "Key Points"
    - Use Trimmomatic to get rid of adapters and low-quality bases or reads.
    - Rerun the FastQC Analysis to check if our sequences have been succesfully filtered and trimmed.