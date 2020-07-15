# intro-to-HPC-course-2020
Introduction to HPC/working on the LRZ linux cluster

This course is an introduction to working in an HPC environment and will cover why we use this and the basics of, such as logging in and submitting jobs.

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

## Refresher: Working on a Unix system 

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

## Access and login 

To access the Linux cluster we will be using ssh (Note: those of you who do not have an account with LRZ will be unable to do this themselves but I will explain how to do this on my machine). 

## First time set-up

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
ssh-keygen -t ecdsa -b 521 -a 100
```

You will be prompted to enter a passphrase , you __must__ enter something, ideally a long sentence which is memorable for you. 

If you set up an account with the LRZ then enter the following 

```
ssh-copy-id -i /home/myaccount/.ssh/id_ecdsa <user>@<targetsystem>
```

If you want you can also run the next command which will allow prompt you to enter your passphrase then subsequently allow you to execute ssh commands without reauthenticating (in the same session, i.e. if you close your terminal then restart you will need to reauthenticate). 

```
ssh-add
```

## Accessing installed software and submitting jobs 




## Reponsible use 


## Rstudio server 

This will likely be the most useful service for many of you. 
