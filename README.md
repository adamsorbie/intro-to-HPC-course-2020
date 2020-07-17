# intro-to-HPC-course-2020
Introduction to HPC/working on the LRZ linux cluster

Some material adapted from: HPC Carpentry - https://hpc-carpentry.github.io/hpc-intro/

This course is an introduction to working in an HPC environment and will cover why we use this and the basics of logging in and submitting jobs.

- [intro-to-HPC-course-2020](#intro-to-hpc-course-2020)
  * [Course Conventions](Course-conventions) 
  * [What is a cluster?](#what-is-a-cluster-)
    + [Difference between cloud computing and a cluster](#difference-between-cloud-computing-and-a-cluster)
    + [Why use a cluster?](#why-use-a-cluster-)
  * [Practical](#practical)
    + [Refresher: Working on a Unix system](#refresher--working-on-a-unix-system)
    + [Finding your way around](#finding-your-way-around)
    + [Access and login](#access-and-login)
    + [First time set-up](#first-time-set-up)
    + [Accessing installed software and submitting jobs](#accessing-installed-software-and-submitting-jobs)
    + [Submitting jobs and writing batch scripts](#submitting-jobs-and-writing-batch-scripts)
    + [Exercise](#exercise)
    + [Uploading files to the cluster](#uploading-files-to-the-cluster)
    + [Downloading files](#downloading-files)
  * [Reponsible use](#reponsible-use)
  * [Rstudio server](#rstudio-server)
  
## Course conventions 

If at any point you have a question, feel free to interrupt and ask. 
<xxx> This nomenclature means replace the text in between the <> with the relevant field for you i.e. <username> == ga92xxxx
The $ sign represents the shell prompt and when shown, means these are commands you should actually enter yourself rather than representing stdout or something else.  

## What is a cluster? 

Very simply, a cluster is a group of computers connected via a very fast network connection and using special software which allows them to work as one. Each individual computers that make up a cluster is termed a node. 

On a typical cluster set-up, there are different types of nodes for performing different types of tasks. For example, when logging in to a cluster from your local machine you will likely on the login node (also called the head node or submit node). This node serves as an access point to the cluster. These nodes are used for uploading and downloading files, setting up software, and running quick tests. __They should never be used for doing actual work__. (You will likely be banned from using the cluster by your system administrator if you try to run a large job on the login node)

The heavy lifting on a cluster is done by the compute (or worker) nodes. There are lots of different types and configurations but generally these nodes have large amounts of memory and powerful CPUs and are set up to perform long running tasks. Here is the configuration of the nodes on the LRZ linux cluster.


https://doku.lrz.de/display/PUBLIC/Job+Processing+on+the+Linux-Cluster


You will not usually work on the compute nodes directly (except in the case of interactive mode which I will explain later). Generally, interaction with these nodes is through  a job scheduler (the scheduler used on the Linux cluster is called SLURM). This software uses an algorithm to optimise the scheduling of jobs and assignment of resources, while also allowing resource monitoring. 

### Difference between cloud computing and a cluster 

You may have heard the term cloud computing. But what's the difference between cloud computing and HPC? At it's most basic, the difference is that HPC represents a physical system where the resources are fixed (i.e. the CoolMUC2 cluster we will be using today) while cloud computing refers more to scalable, on-demand computing, for example AWS or Google cloud. That is, the resources are not necessarily fixed and can be increased or decreased depending on your requirements. Of course, there are other differences too but for now this is the main difference that matters for us.  


### Why use a cluster?

For analysis of most of the biological data we generate in this lab at the moment we don't really need a cluster. However, as sequencing gets cheaper and cheaper and more commonplace it will probably become a larger part of the data we produce. Right now, most of us have at least one set of sequencing data, whether that is 16S, Shotgun or RNAseq. 

The majority of 16S analysis can be performd without utilising HPC but that does not mean that it isn't useful for this purpose (I will provide an example of how it can be useful at the end of this course).  Most of the laptops or PCs we have will have a maximum of 16GB RAM and a 3 or 4GhZ quad-core processor. This is a lot for everyday use but simply isn't enough to work with huge datasets generated by NGS or run some tools for analysis. As an example, I often run PICRUSt2, which requires an absolute minimum of 16GB, around the upper limits of our personal machines. 

Shotgun and RNAseq in most cases, requires access to very powerful hardware as datasets can be tens to hundreds of GBs in size. Of course you can always outsource this but it's good to at least have an understanding of the process and if you are able to perform these analyses yourself it takes less time to get the data and figures. 


## Practical 

### Refresher: Working on a Unix system 

Hopefully you have taken (or at least had a quick look at) the previous course I did: https://github.com/adamsorbie/unix_shell_course-2020-02-14

If not we will do a very short refresher on working on a Unix system. 

### Finding your way around 

Print the working directory 

```
$ pwd
```
Example output on my machine:

```
/home/adamsorbie
``` 

List files and folders

```
$ ls
``` 

Example output:

```
bin/ analyses/ Documents/ 
```
Creating new directories (this might be necessary if you have a new installation of WSL) 

```
$ mkdir test
```
Check it worked: 

```
$ ls 
``` 

Expected output: 

```
test/
``` 

Moving around the file system 

```
$ cd bin/ 
```

Test with pwd 
```
$ pwd
``` 
Output on my machine:

``` 
/home/adamsorbie/bin
```
Play around with these examples and those in the linked course for 5 minutes or so until you feel comfortable with these concepts. 

### Access and login 

To access the Linux cluster we will be using ssh (Note: those of you who do not have an account with LRZ will be unable to do this themselves but I will explain how to do this on my machine). 

### First time set-up

These are the nodes you are likely to use to login. 

| Command  | Login Node |
|---------------|-------|
| ssh -Y lxlogin1.lrz.de -l <user> | Haswell (CoolMUC-2) login node |
| ssh -Y lxlogin2.lrz.de -l <user> | Haswell (CoolMUC-2) login node |
| ssh -Y lxlogin3.lrz.de -l <user> | Haswell (CoolMUC-2) login node |
| ssh -Y lxlogin4.lrz.de -l <user> | Haswell (CoolMUC-2) login node |

Important info from LRZ!!!!! 

"The login nodes are meant for preparing your jobs, developing your programs, and as a gateway for copying data from your own computer to the cluster and back again. Since this resource is shared among many users, LRZ requires that you do not start any long-running or memory-hogging programs on these nodes; production runs should use batch jobs that are submitted to the SLURM scheduler. Our SLURM configuration also supports semi-interactive testing. Violation of the usage restrictions on the login nodes may lead to your account being blocked from further access to the cluster, apart from your processes being forcibly removed by LRZ administrative staff!"

Additionally, the following steps are super important to make sure your connection is secure. Multiple HPC centres across Europe were hacked recently, including the LRZ so it's important to follow these steps carefully. 

First of all we need to generate an SSH-key to be able to access the LRZ system from outside. 

Run the following command and press enter

```
$ ssh-keygen -t ecdsa -b 521 -a 100
```

You will be prompted to enter a passphrase , you __must__ enter something, ideally a long sentence which is memorable for you. 

If you set up an account with the LRZ then enter the following 

```
$ ssh-copy-id -i /home/myaccount/.ssh/id_ecdsa <user>@<targetsystem>
```

If you want you can also run the next command which will allow prompt you to enter your passphrase then subsequently allow you to execute ssh commands without reauthenticating (in the same session, i.e. if you close your terminal then restart you will need to reauthenticate). 

```
$ ssh-add
```

Those of you who have access should login to the cluster now. 

### Accessing installed software and submitting jobs 

On high-performance computing systems, it is often the case that no software is loaded by default. If we want to use a particular package, we will usually need to “load” it ourselves.

Before we start using individual software packages, it's important to understand the reasoning behind this approach. The three biggest factors are:

software incompatibilities
versioning
dependencies


Software incompatibility is a major problem for programmers and bioinformaticians alike. Sometimes the presence (or absence) of a package will break others that depend on it.  

Software versioning is another common issue. You might depend on a particular version of a package for an analysis you are doing - if the software version changes (for instance, if a package was updated), it might affect your results. This sort of thing happens all the time and can be especially problematic in R. Having access to multiple software versions allows you to prevent issues with software versioning from affecting your results.

Dependencies are where a particular software package (or even a particular version) depends on having access to another software package (or even a particular version of another software package). If the package your package depends on is not there or is the wrong version, then it may not work. 

The solution to this on HPC systems is Environment modules. These are software packages which allow you to dynamically modify the environment you are working in. We need to take a little quick detour to understand this better. When you type a command on a unix system (excluding builtins like cd and echo) the shell searches for it in your path. If it finds it then the command will run, otherwise the shell will return an error. 

To see what is currently in your path variable type the following command:

```
$ echo $PATH
```

Since hundreds or thousands of researchers are using the Linux cluster and all have different software needs, it would be really messy for everything to be installed and available at once, the output of ```$ echo $PATH``` would be pages and pages long. Environment modules manage this problem by dynamically adding packages to your path making sure you only have what you need. 

When logging to the linux cluster you start with only a few modules preloaded. For those of us who have access to the cluster, let's login now and explore this a little further. 

Type the following command to see what these are: 

```
$ module list
```

You should see something like the following:

```
Currently Loaded Modulefiles:
 1) admin/1.0     3) lrz/1.0                5) intel/19.0.5           7) intel-mpi/2019.7.217
 2) tempdir/1.0   4) spack/staging/20.1.1   6) intel-mkl/2019.5.281
```

You can also type ``` $ echo $PATH``` to check what's already in your path variable. 

To see what modules are available we can type: 

```
$ module avail
```

You will notice there is a few pages of packages listed, including different versions of the same software. Let's go ahead and load python. 

```
$ module load python
```

Note: There are multiple versions of python installed, if you do not specify a version it will just load the default. 

Now let's check our path to see if python is now included there (here we pipe the output of echo to grep and search for python to make it easier to see)

```
$ echo $PATH | grep python
```

To unload a package we can type ```module unload <module>``` Let's unload python. 

```
$ module unload python
```


In the last few years, some effors have been made to manage packages a little better. These efforts are mostly focused on non-HPC systems but will still be useful for you if you are working on the cluster and need to install something that isn't available in the listed modules. Since you obviously don't have administrator (Root on unix systems) privileges you generally can't install packages in the default location. To get around we can use the package manager conda. Conda is software tool which allows you create separate environments yourself and manage versions and dependencies. It's not perfect but it definitely makes managing packages a lot easier. 

We will install and create a PICRUSt2 environment here as an example. 

First we need to load the python module so we have access to conda 

```
$ module load python
```

Install PICRUSt2 using conda 

```
$ conda create -n picrust2 -c bioconda -c conda-forge picrust2=2.3.0_b python=3.6
```

To activate your new environment run 

```
$ source activate picrust2
```

Note: On newer versions of conda the command is ``$ conda activate <env>```

In my experience, most packages I need to use are not available as modules and need to be installed via conda. 

### Submitting jobs and writing batch scripts

Let's move on now to submitting jobs using the SLURM scheduler. We will start with a very simple bash script to understand how to submit a job and the commands we can use to check the status of or cancel a job.

### Exercise 

We are going to submit a very simple test script to see how this process works in practice. 

Firstly, clone the course repository with the following command: 

```
$ git clone https://github.com/adamsorbie/intro-to-HPC-course-2020.git
```

Note: Git should be available be default on most unix systems so you do not need to load a module or install anything here. 

Now, let's run this scrip on the login node to see what it does. 

```
$ bash example-job.sh 
```
 
 What was your output? 
 
 
 
 You should see something like this: 
 
```
This is script is running on cm2login1
```

The ```hostname```command just tells which computer you are working. You can see that we are working on one of the login nodes. 

Here we should also quickly discuss the distinction between what can be run on a login node and what cannot. Small simple Bash/Python/R scripts like this are fine because they are not long running and barely require any memory. If you need to test something which might require a large amount of memory or takes longer than ~5 minutes to run then there is also a semi-interactive mode which you can use for this. 

Now we are going to submit our job to the cluster. To do so, run the following: 

```
sbatch example-job.sh
```

We can check the status by using the ```squeue``` command like so:

```
$ squeue -u <username>
```

We can cancel our job using ```scancel```

```
$ scancel <jobid>
```

Now we will resubmit, letting it run this time. It might take a little while to run because there will be other jobs in the queue but once finished you should see a file called slurm-xxxxxx.out where xxxxxx will be replaced by some numbers. If you look at this file you should see the output of our bash script. You will also notice the output is different this time because this script was ran on a compute node and not the login node. 

The script we ran above was very basic and we didn't specify any parameters so it just ran with the defaults. When we are conducting an analysis or running some software this is not what we want. In most use cases (at least for us), submitted scripts resemble still bash scripts but have a few extra few lines at the top which tell SLURM which cluster and partition to use and how much memory you need etc. Below is an annotated example of a script I wrote recently to run RAXML, a program for inference of phylogenetic trees. 

We won't actually write and submit our own 'real' jobs today but let's go through this script line by line so you can understand what everything does. Before running your own scripts you should refer to LRZ [documentation](https://doku.lrz.de/display/PUBLIC/Running+serial+jobs+on+the+Linux-Cluster)

```
#!/bin/bash 

## These are the extra lines required for the SLURM scheduler
#SBATCH -J "raxml_ref_tree" ## job name
#SBATCH -D ./ ## working directory
#SBATCH -o ./%x.%j.%N.out ## writes out error logs, %x encodes job name, %j job ID and %N the master node 
#SBATCH --get-user-env ## Sets the user environment properly
#SBATCH --clusters=serial ## cluster to run the job on
#SBATCH --partition=serial_std ## which partition of the above cluster to use (each cluster has different partitions for different use cases) 
#SBATCH --mem=50gb ## amount of memory you need to run your job 
#SBATCH --cpus-per-task=4 ## number of cpus
#SBATCH --mail-type=end ## sends you an email when your job is done
#SBATCH --mail-user=adam.sorbie@tum.de ## this is really important, you must enter a valid email address here! 
#SBATCH --export=NONE ## whether to export the user environment or not 
#SBATCH --time=00:30:00 ## time to reserve 

## set number of threads
export OMP_NUM_THREADS=4

## load modules 
module load python

## activate conda env required to run job 
source activate metag_prof

## code goes below
cd input || exit

dirs=$(ls -d */)

for i in $dirs;
do
    cd $i
    outname=$(echo $i | cut -f1 -d "_")
    raxmlHPC-PTHREADS-SSE3 -T 4 -f E -p 1234 -x 5678 -m GTRGAMMA -N 1000 -s *.fna -n ${outname}_raxml_tree_GTRG

    # test code
    # check alignment can be read and doesn't contain any duplicate characters etc
    #raxml-ng --check --msa *.fna --model GTR+G --prefix T1
    # run raxml-ng
    #raxml-ng --all --bootstrap --msa *.fna --model GTR+G --tree pars{25}, rand{25} --threads 4 --seed 42 --bs-trees 1000
    raxml-ng --evaluate --seed 1234 --log progress --threads 4 --msa *.fna --model GTR+G --tree RAxML_fastTree.${outname}_raxml_tree_GTRG --brlen scaled --prefix GTRG
    cd ..
done
```
### Uploading files to the cluster 

You might be wondering after looking through this how we get files onto the cluster to conduct our analyses. For most use cases, we can use the command line tool ```scp``` short for secure copy protocol. It used like so:

```
$ scp file username@login.de:/home/user/data
```

We need the file path of our home directory to know where to copy to so go ahead and use ```pwd```to find this out. Now let's log out (ctrl-D) and copy a file from our local system to the cluster. 

If you only recently installed the unix system you are working on you probably don't have any files yet so we firstly need to create one. 

Create a file using the ```touch ``` command called test_upload.txt

```
touch test_upload.txt 
```

We will also add some text to the file

```
echo "This is a test file for uploading" >> test_upload.txt
```

Now we're going to try copying this file onto the cluster. Using the same syntax as above, we can copy the file like so (replace the file path with your pwd output):

```
scp test.txt <username>@lxlogin1.lrz.de:/dss/dsshome1/lxc##/<user_folder> 
```

### Downloading files 

Downloading files follows the same principles. Log back on to the cluster and firstly check if the file we just uploaded is there using ```ls```
If you cloned the repository correctly earlier you should also see a file in the intro-to-HPC-course folder called test_download.txt. If yes, we can log out again and proceed with downloading our test file. 

Now download the test_download.txt file using the following command: 

```
scp <username>@lxlogin1.lrz.de:/dss/dsshome1/lxc##/<user_folder>/intro-to-HPC-course/test_download.txt . 
```

It's basically the same thing reversed. The ```.``` is the target and just means current directory. 

## Reponsible use 

Key Points

Again do not run batch jobs on the login node. Never. 
Although in our case we do not pay for our usage, be mindful of the amount of compute time you request. The LRZ has guidelines [here](https://doku.lrz.de/display/PUBLIC/Guidelines+for+resource+selection). Even after reading these it still might be difficult to figure out how much memory or time you need. 
Unless you have ran a job a few times it's very difficult to know how much you will need. What you can do however is when initially submiting job you can allocate more time than you think you will need just in case then use the output to check memory and time and edit your subsequent submissions accordingly you can do this with the ```sacct``` command. 

``` 
sacct -M clustername -j JOBID -o jobid, partition, user, start, end, elapsed, maxrss

```

Your data on the system is your responsibility, user directories are backed up but you only have a limit of 100gb here so if you are working with lots of data or very large files then you will need to use the SCRATCH file system. This should only really be used temporarily as it is not backed up and files on here are regularly deleted by the system administrators. 

If you have to upload many files e.g. from a sequencing experiment, it's good practice to compress them into one archive and upload rather than uploading each file individually. 

## Rstudio server 

This will likely be the most useful and familiar cluster-based service for many of you. The LRZ operates an Rstudio Server instance running on the linux cluster, which allows you to use the power of the cluster while working in a more familiar environment. Given that most of you will be somewhat familiar with Rstudio already it doesn't need much of an introduction and there are only minor differences between using Rstudio here compared to on your own computer. 

You can login to this system [here](https://www.rstudio.lrz.de/) (if you have a valid LRZ account) 
