# Containers in HPC 

## Presented in the Petascale Computing Institute 2019

This page is meant as a quick reference for the commands presented during the 
Petascale Institute, and to facilitate cutting-and-pasting where necessary.
This page is not meant to be a standalone reference guide for using containers
in an HPC environment.

[https://bluewaters.ncsa.illinois.edu/petascale-computing-2019](https://bluewaters.ncsa.illinois.edu/petascale-computing-2019)

<br>


### Part 1: Docker on Local Machine

Installation:

[https://docs.docker.com/docker-for-windows/install/]

[https://docs.docker.com/docker-for-mac/install/]

[https://docs.docker.com/install/linux/docker-ce/ubuntu/]

[https://docs.docker.com/install/linux/docker-ce/centos/]

[https://training.play-with-docker.com/beginner-linux/]


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

<br>


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

<br>

### Part 3a: Containers on Stampede2

Login:
```
$ ssh USER@stampede2.tacc.utexas.edu
```

Stage the data:
```
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
```

Pull the container:
```
[login]$ idev 
...
[compute]$ module load tacc-singularity python3
[compute]$ singularity pull --name wallen-fastqc-0.11.7.simg docker://wallen/fastqc:0.11.7
[compute]$ ls $WORK/singularity_cache/
```

Prepare the job script and submit the job:
```
[login]$ cat singularity_job.slurm
#!/bin/bash
#SBATCH -J myjob
#SBATCH -o myjob.o%j
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -t 00:10:00
#SBATCH -p skx-dev
#SBATCH -A myalloc   # Allocation name

module load tacc-singularity

SIMG=$WORK/singularity_cache/wallen-fastqc-0.11.7.simg

singularity exec $SIMG fastqc SP1.fq

[login]$ sbatch singularity_job.slurm
...
Submitted batch job 4197252
```

<br>

### Part 3b: Containers on Blue Waters

Login:
```
$ ssh USER@bwbay.ncsa.illinois.edu
```

Stage the data:
```
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
```

Pull the container:
```
[login]$ qsub -I -l nodes=1:ppn=1 -l walltime=00:30:00
...
[compute]$ module load shifter
[compute]$ shifterimg pull docker:wallen/fastqc:0.11.7
2019-08-19T15:44:49 Pulling Image: docker:wallen/fastqc:0.11.7, status: READY
[compute]$ shifterimg images | grep fastqc
bluewaters docker     READY    6d2726df2e   2019-08-19T15:44:14 wallen/fastqc:0.11.7
```

Prepare the job script and submit the job:
```
[login]$ cat shifter_job.pbs
#!/bin/bash
#PBS -N testjob
#PBS -e $PBS_JOBID.err
#PBS -o $PBS_JOBID.out
#PBS -l nodes=1:ppn=1:xe
#PBS -l walltime=00:10:00
#PBS -A myalloc
#PBS -l gres=shifter16

module load shifter
IMG=docker:wallen/fastqc:0.11.7

aprun -b shifter --image=$IMG fastqc SP1.fq

[login]$ qsub shifter_job.pbs
INFO: Job submitted to account: myalloc
10240521.bw
```

<br>



### Part 3c: Containers on Cori

Login:
```
$ ssh USER@cori.nersc.gov
```

Stage the data:
```
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
```

Pull the container:
```
[login]$ salloc -N 1 -C haswell -q interactive -t 00:30:00
...
[compute]$ shifterimg pull docker:wallen/fastqc:0.11.7
2019-08-19T14:02:58 Pulling Image: docker:wallen/fastqc:0.11.7, status: READY
[compute]$ shifterimg images | grep fastqc
cori       docker     READY    6d2726df2e   2019-08-19T14:02:57 wallen/fastqc:0.11.7
```

Prepare the job script and submit the job:
```
[login]$ cat shifter_job.slurm 
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --time=00:10:00
#SBATCH --qos=debug
#SBATCH --constraint=haswell
#SBATCH --image=docker:wallen/fastqc:0.11.7

srun -n 1 shifter fastqc SP1.fq

[login]$ sbatch shifter_job.slurm
Submitted batch job 24000106
```

<br>


### Part 4: Reference

Docker:

[https://www.docker.com/]

[https://docs.docker.com/get-started/]

[https://training.play-with-docker.com/beginner-linux/]


Singularity:

[https://singularity.lbl.gov/]

[https://sylabs.io/]


Shifter:

[https://github.com/NERSC/shifter]

[https://docs.nersc.gov/programming/shifter/how-to-use/]

[https://bluewaters.ncsa.illinois.edu/shifter]


Containers:

[https://hub.docker.com/]

[https://singularity-hub.org/]

[https://biocontainers.pro/]

Sample data:

Jay Hesselberth

Genome Analysis Workshop

https://molb7621.github.io/workshop/index.html 


Questions:

Joe Allen | wallen@tacc.utexas.edu











