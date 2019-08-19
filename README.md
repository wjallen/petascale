# Containers in HPC 

## Presented in the Petascale Computing Institute 2019

This page is meant as a quick reference for the commands presented during the 
Petascale Institute, and to facilitate cutting-and-pasting where necessary.
This page is not meant to be a standalone reference guide for using containers
in an HPC environment.

https://bluewaters.ncsa.illinois.edu/petascale-computing-2019


### Part 1: Docker on Local Machine

Installation:

https://docs.docker.com/docker-for-windows/install/

https://docs.docker.com/docker-for-mac/install/

https://docs.docker.com/install/linux/docker-ce/ubuntu/

https://docs.docker.com/install/linux/docker-ce/centos/

Basic commands:
```
$ docker version      # show version information

$ docker images       # show images you have pulled

$ docker ps           # show running containers

$ docker run hello-world       

... a slightly better way to do it:

$ docker pull hello-world:latest

$ docker images
REPOSITORY        TAG         IMAGE ID        CREATED         SIZE
hello-world       latest      fce289e99eb9    7 months ago    1.84kB

$ docker run hello-world:latest

$ docker inspect hello-world
```

Real-world example:
```
$ docker pull biocontainers/fastqc:v0.11.5_cv4

$ docker run --rm biocontainers/fastqc:v0.11.5_cv4 fastqc --help
```

Interactive example:
```
$ docker run --rm -it biocontainers/fastqc:v0.11.5_cv4 /bin/bash

[container]$ pwd
/data

[container]$ whoami
biodocker

[container]$ which fastqc
/usr/local/bin/fastqc

[container]$ fastqc --help
```


### Part 2: Develop your own Container

The Dockerfile:
```
$ pwd 
/Users/username/fastqc-dev-folder

$ ls
Dockerfile

$ cat Dockerfile
FROM ubuntu:16.04

RUN apt-get update && apt-get upgrade -y \
    && apt-get install -y default-jre perl wget zip

RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.7.zip \
    && unzip fastqc_v0.11.7.zip \
    && rm fastqc_v0.11.7.zip \
    && chmod +x /FastQC/fastqc 

ENV PATH "/FastQC:$PATH"
```

Build and push (replace `username` with your Docker Hub username):
```
$ docker build -t username/fastqc:0.11.7 ./

$ docker images
REPOSITORY         TAG         IMAGE ID          CREATED             SIZE
username/fastqc    0.11.7      2005acfb2869      16 minutes ago      460MB
hello-world        latest      fce289e99eb9      7 months ago        1.84kB

$ docker run --rm username/fastqc:0.11.7 which fastqc
/FastQC/fastqc

$ docker push username/fastqc:0.11.7
```

Getting more help:
```
$ docker --help            # show all docker options and summaries

$ docker COMMAND --help    # show options and summaries for a particular 
                           # command
```


### Part 3a: Containers on Stampede2

```
# Login

$ ssh USER@stampede2.tacc.utexas.edu

# Stage the data
[login]$ wget https://wjallen.github.io/petascale/SP1.fq
[login]$ head SP1.fq
@@cluster_2:UMI_ATTCCG
TTTCCGGGGCACATAATCTTCAGCCGGGCGC
+
9C;=;=<9@4868>9:67AA<9>65<=>591
@cluster_8:UMI_CTTTGA
TATCCTTGCAATACTCTCCGAACGGGAGAGC
+
1/04.72,(003,-2-22+00-12./.-.4-
@cluster_12:UMI_GGTCAA
GCAGTTTAAGATCATTTTATTGAAGAGCAAG

# Pull the container
[login]$ idev 
...
[compute]$ module load tacc-singularity python3
[compute]$ singularity pull --name wallen-fastqc-0.11.7.simg docker://wallen/fastqc:0.11.7
[compute]$ ls $WORK/singularity_cache/

# Prepare the job script and submit the job
[login]$ cat singularity_job.slurm
#!/bin/bash
#SBATCH -J myjob
#SBATCH -o myjob.o%j
#SBATCH -e myjob.e%j
#SBATCH -p skx-dev
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -t 00:10:00
#SBATCH -A myproject   # Allocation name

module load tacc-singularity
module swap python2 python3

SIMG=$WORK/singularity_cache/wallen-fastqc-0.11.7.simg

singularity exec $SIMG fastqc SP1.fq

[login]$ sbatch singularity_job.slurm
```

### Part 3a: Containers on Blue Waters

### Part 3a: Containers on Cori







### Part 4: Reference

links down here
