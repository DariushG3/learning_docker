
Table of Contents
=================

   * [Learning Docker](#learning-docker)
      * [Introduction](#introduction)
      * [Installing the Docker Engine](#installing-the-docker-engine)
      * [Checking your installation](#checking-your-installation)
      * [Basics](#basics)
      * [Dockerfile](#dockerfile)
      * [CMD](#cmd)
      * [ENTRYPOINT](#entrypoint)
      * [Building an image](#building-an-image)
      * [Renaming an image](#renaming-an-image)
      * [Running an image](#running-an-image)
      * [Resource usage](#resource-usage)
      * [Copying files between host and container](#copying-files-between-host-and-container)
      * [Sharing between host and container](#sharing-between-host-and-container)
         * [File permissions](#file-permissions)
         * [File Permissions 2](#file-permissions-2)
         * [Read only](#read-only)
      * [Removing the image](#removing-the-image)
      * [Committing changes](#committing-changes)
      * [Access running container](#access-running-container)
      * [Cleaning up exited containers](#cleaning-up-exited-containers)
      * [Installing Perl modules](#installing-perl-modules)
      * [Creating a data container](#creating-a-data-container)
      * [R](#r)
      * [Saving and transferring a Docker image](#saving-and-transferring-a-docker-image)
      * [Pushing to Docker Hub](#pushing-to-docker-hub)
      * [Tips](#tips)
      * [Useful links](#useful-links)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

Sat 14 Aug 2021 21:24:03 JST

Learning Docker
================

## Introduction

Docker is an open source project that allows one to pack, ship, and run any application as a lightweight container. An analogy of Docker containers are shipping containers, which provide a standard and consistent way of shipping just about anything. The container includes everything that is needed for an application to run including the code, system tools, and the necessary dependencies. If you wanted to test an application, all you need to do is to download the Docker image and run it in a new container. No more compiling and installing missing dependencies!

The [overview](https://docs.docker.com/get-started/overview/) at <https://docs.docker.com/> provides more information. For more a more hands-on approach, check out know [Enough Docker to be Dangerous](https://docs.docker.com/) and [this short workshop](https://davetang.github.io/reproducible_bioinformatics/docker.html) that I prepared for BioC Asia 2019.

This README was generated from the R Markdown file `readme.Rmd`, which can executed via the `create_readme.sh` script.

## Installing the Docker Engine

To get started, you will need to install the Docker Engine; check out [this guide](https://docs.docker.com/engine/install/).

## Checking your installation

To see if everything is working, try to obtain the Docker version.

``` bash
docker --version
```

    ## Docker version 20.10.7, build f0df350

And run the `hello-world` image. (The `--rm` parameter is used to automatically remove the container when it exits.)

``` bash
docker run --rm hello-world
```

    ## 
    ## Hello from Docker!
    ## This message shows that your installation appears to be working correctly.
    ## 
    ## To generate this message, Docker took the following steps:
    ##  1. The Docker client contacted the Docker daemon.
    ##  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    ##     (amd64)
    ##  3. The Docker daemon created a new container from that image which runs the
    ##     executable that produces the output you are currently reading.
    ##  4. The Docker daemon streamed that output to the Docker client, which sent it
    ##     to your terminal.
    ## 
    ## To try something more ambitious, you can run an Ubuntu container with:
    ##  $ docker run -it ubuntu bash
    ## 
    ## Share images, automate workflows, and more with a free Docker ID:
    ##  https://hub.docker.com/
    ## 
    ## For more examples and ideas, visit:
    ##  https://docs.docker.com/engine/userguide/

## Basics

The two guides linked in the introduction section provide some information on the basic commands but I'll include some here as well. One of the main reasons I use Docker is for building tools. For this purpose, I use Docker like a virtual machine, where I can install whatever I want. This is important because I can do my testing in an isolated environment and not worry about affecting the main server. I like to use Ubuntu because it's a popular Linux distribution and therefore whenever I run into a problem, chances are higher that someone else has had the same problem, asked a question on a forum, and received a solution.

Before we can run Ubuntu using Docker, we need an image. We can obtain an Ubuntu image from the [official Ubuntu image repository](https://hub.docker.com/_/ubuntu/) from Docker Hub by running `docker pull`.

``` bash
docker pull ubuntu:18.04
```

    ## 18.04: Pulling from library/ubuntu
    ## Digest: sha256:7bd7a9ca99f868bf69c4b6212f64f2af8e243f97ba13abb3e641e03a7ceb59e8
    ## Status: Image is up to date for ubuntu:18.04
    ## docker.io/library/ubuntu:18.04

To run Ubuntu using Docker, we use `docker run`.

``` bash
docker run --rm ubuntu:18.04 cat /etc/os-release
```

    ## NAME="Ubuntu"
    ## VERSION="18.04.5 LTS (Bionic Beaver)"
    ## ID=ubuntu
    ## ID_LIKE=debian
    ## PRETTY_NAME="Ubuntu 18.04.5 LTS"
    ## VERSION_ID="18.04"
    ## HOME_URL="https://www.ubuntu.com/"
    ## SUPPORT_URL="https://help.ubuntu.com/"
    ## BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
    ## PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
    ## VERSION_CODENAME=bionic
    ## UBUNTU_CODENAME=bionic

You can work interactively with the Ubuntu image by specifying the `-it` option.

``` bash
docker run --rm -it ubuntu:18:04 /bin/bash
```

You may have noticed that I keep using the `--rm` option, which removes the container once you quit. If you don't use this option, the container is saved up until the point that you exit; all changes you made, files you created, etc. are saved. Why am I deleting all my changes? Because there is a better (and more reproducible) way to make changes to the system and that is by using a Dockerfile.

## Dockerfile

A Dockerfile is a text file that contains instructions for building Docker images. A Dockerfile adheres to a specific format and set of instructions, which you can find at [Dockerfile reference](https://docs.docker.com/engine/reference/builder/). There is also a [Best practices guide](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) for writing Dockerfiles.

I have an example Dockerfile that uses the Ubuntu 18.04 image to build [BWA](https://github.com/lh3/bwa), a popular short read alignment tool used in bioinformatics.

``` bash
cat Dockerfile
```

    ## FROM ubuntu:18.04
    ## 
    ## MAINTAINER Dave Tang <me@davetang.org>
    ## LABEL source="https://github.com/davetang/learning_docker/blob/master/Dockerfile"
    ## 
    ## RUN apt-get clean all && \
    ##     apt-get update && \
    ##     apt-get upgrade -y && \
    ##     apt-get install -y \
    ##      build-essential \
    ##      wget \
    ##      zlib1g-dev && \
    ##     apt-get clean all && \
    ##     apt-get purge && \
    ##     rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
    ## 
    ## RUN mkdir /src && \
    ##     cd /src && \
    ##     wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2 && \
    ##     tar xjf bwa-0.7.17.tar.bz2 && \
    ##     cd bwa-0.7.17 && \
    ##     make && \
    ##     mv bwa /usr/local/bin && \
    ##     cd && rm -rf /src
    ## 
    ## WORKDIR /work
    ## 
    ## CMD ["bwa"]

## CMD

The [CMD](https://docs.docker.com/engine/reference/builder/#cmd) instruction in a Dockerfile does not execute anything at build time but specifies the intended command for the image; there can only be one CMD instruction in a Dockerfile and if you list more than one CMD then only the last CMD will take effect. The main purpose of a CMD is to provide defaults for an executing container.

## ENTRYPOINT

An [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint) allows you to configure a container that will run as an executable. ENTRYPOINT has two forms:

-   ENTRYPOINT \["executable", "param1", "param2"\] (exec form, preferred)
-   ENTRYPOINT command param1 param2 (shell form)

``` bash
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

Use `--entrypoint` to override ENTRYPOINT instruction.

``` bash
docker run --entrypoint
```

## Building an image

Use the `build` subcommand to build Docker images and use the `-f` parameter if your Dockerfile is named as something else otherwise Docker will look for a file named `Dockerfile`. The period at the end, tells Docker to look in the current directory.

``` bash
cat build.sh
```

    ## #!/usr/bin/env bash
    ## 
    ## set -euo pipefail
    ## 
    ## ver=0.7.17
    ## 
    ## docker build -t davetang/bwa:${ver} .

You can push the built image to [Docker Hub](https://hub.docker.com/) if you have an account. I have used my Docker Hub account name to name my Docker image.

``` bash
# use -f to specify the Dockerfile to use
# the period indicates that the Dockerfile is in the current directory
docker build -f Dockerfile.base -t davetang/base .

# log into Docker Hub
docker login

# push to Docker Hub
docker push davetang/base
```

## Renaming an image

Use `docker image tag`.

``` bash
docker image tag old_image_name:latest new_image_name:latest
```

## Running an image

[Docker run documentation](https://docs.docker.com/engine/reference/run/).

``` bash
docker run --rm davetang/bwa:0.7.17
```

    ## 
    ## Program: bwa (alignment via Burrows-Wheeler transformation)
    ## Version: 0.7.17-r1188
    ## Contact: Heng Li <lh3@sanger.ac.uk>
    ## 
    ## Usage:   bwa <command> [options]
    ## 
    ## Command: index         index sequences in the FASTA format
    ##          mem           BWA-MEM algorithm
    ##          fastmap       identify super-maximal exact matches
    ##          pemerge       merge overlapping paired ends (EXPERIMENTAL)
    ##          aln           gapped/ungapped alignment
    ##          samse         generate alignment (single ended)
    ##          sampe         generate alignment (paired ended)
    ##          bwasw         BWA-SW for long queries
    ## 
    ##          shm           manage indices in shared memory
    ##          fa2pac        convert FASTA to PAC format
    ##          pac2bwt       generate BWT from PAC
    ##          pac2bwtgen    alternative algorithm for generating BWT
    ##          bwtupdate     update .bwt to the new format
    ##          bwt2sa        generate SA from BWT and Occ
    ## 
    ## Note: To use BWA, you need to first index the genome with `bwa index'.
    ##       There are three alignment algorithms in BWA: `mem', `bwasw', and
    ##       `aln/samse/sampe'. If you are not sure which to use, try `bwa mem'
    ##       first. Please `man ./bwa.1' for the manual.

## Resource usage

To [restrict](https://docs.docker.com/config/containers/resource_constraints/) CPU usage use `--cpus=n` and use `--memory=` to restrict the maximum amount of memory the container can use.

We can confirm the limited CPU usage by running an endless while loop and using `docker stats` to confirm the CPU usage. *Remember to use `docker stop` to stop the container after confirming the usage!*

Restrict to 1 CPU.

``` bash
# run in detached mode
docker run --rm -d --cpus=1 davetang/bwa:0.7.17 perl -le 'while(1){ }'

# check stats and use control+c to exit
docker stats
CONTAINER ID   NAME             CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
8cc20bcfa4f4   vigorous_khorana   100.59%   572KiB / 1.941GiB   0.03%     736B / 0B   0B / 0B     1

docker stop 8cc20bcfa4f4
```

Restrict to 1/2 CPU.

``` bash
# run in detached mode
docker run --rm -d --cpus=0.5 davetang/bwa:0.7.17 perl -le 'while(1){ }'

# check stats and use control+c to exit
docker stats

CONTAINER ID   NAME             CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
af6e812a94da   unruffled_liskov   50.49%    584KiB / 1.941GiB   0.03%     736B / 0B   0B / 0B     1

docker stop af6e812a94da
```

## Copying files between host and container

Use `docker cp` but I recommend mounting a volume to a Docker container (see next section).

``` bash
docker cp --help

Usage:  docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
        docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

Copy files/folders between a container and the local filesystem

Options:
  -L, --follow-link   Always follow symbol link in SRC_PATH
      --help          Print usage

# find container name
docker ps -a

# create file to transfer
echo hi > hi.txt

docker cp hi.txt fee424ef6bf0:/root/

# start container
docker start -ai fee424ef6bf0

# inside container
cat /root/hi.txt 
hi

# create file inside container
echo bye > /root/bye.txt
exit

# transfer file from container to host
docker cp fee424ef6bf0:/root/bye.txt .

cat bye.txt 
bye
```

## Sharing between host and container

Use the `-v` flag to mount directories to a container so that you can share files between the host and container.

In the example below, I am mounting `data` from the current directory (using the Unix command `pwd`) to `/work` in the container. I am working from the root directory of this GitHub repository, which contains the `data` directory.

``` bash
ls data
```

    ## README.md
    ## chrI.fa.gz

Any output written to `/work` inside the container, will be accessible inside `data` on the host. The command below will create BWA index files for `data/chrI.fa.gz`.

``` bash
docker run --rm -v $(pwd)/data:/work davetang/bwa:0.7.17 bwa index chrI.fa.gz
```

    ## [bwa_index] Pack FASTA... 0.28 sec
    ## [bwa_index] Construct BWT for the packed sequence...
    ## [bwa_index] 10.82 seconds elapse.
    ## [bwa_index] Update BWT... 0.09 sec
    ## [bwa_index] Pack forward-only FASTA... 0.16 sec
    ## [bwa_index] Construct SA from BWT and Occ... 3.98 sec
    ## [main] Version: 0.7.17-r1188
    ## [main] CMD: bwa index chrI.fa.gz
    ## [main] Real time: 15.817 sec; CPU: 15.396 sec

We can see the newly created index files.

``` bash
ls -lrt data
```

    ## total 63416
    ## -rw-r--r--@ 1 dtang  staff   4772981 12 Sep  2015 chrI.fa.gz
    ## -rw-r--r--  1 dtang  staff       194 14 Aug 11:50 README.md
    ## -rw-r--r--  1 dtang  staff  15072516 14 Aug 21:23 chrI.fa.gz.bwt
    ## -rw-r--r--  1 dtang  staff   3768110 14 Aug 21:23 chrI.fa.gz.pac
    ## -rw-r--r--  1 dtang  staff        41 14 Aug 21:23 chrI.fa.gz.ann
    ## -rw-r--r--  1 dtang  staff        13 14 Aug 21:23 chrI.fa.gz.amb
    ## -rw-r--r--  1 dtang  staff   7536272 14 Aug 21:23 chrI.fa.gz.sa

Remove the index files, since we no longer need them

``` bash
rm data/chrI.fa.gz.*
```

### File permissions

On newer version of Docker, you no longer have to worry about this. However, if you find that the file created inside your container on a mounted volume are owned by `root`, read on.

The files created inside the Docker container will be owned by root; inside the Docker container, you are `root` and the files you produce will have `root` permissions.

``` bash
ls -lrt
total 2816
-rw-r--r-- 1 1211 1211 1000015 Apr 27 02:00 ref.fa
-rw-r--r-- 1 1211 1211   21478 Apr 27 02:00 l100_n100_d400_31_2.fq
-rw-r--r-- 1 1211 1211   21478 Apr 27 02:00 l100_n100_d400_31_1.fq
-rw-r--r-- 1 1211 1211     119 Apr 27 02:01 run.sh
-rw-r--r-- 1 root root 1000072 Apr 27 02:03 ref.fa.bwt
-rw-r--r-- 1 root root  250002 Apr 27 02:03 ref.fa.pac
-rw-r--r-- 1 root root      40 Apr 27 02:03 ref.fa.ann
-rw-r--r-- 1 root root      12 Apr 27 02:03 ref.fa.amb
-rw-r--r-- 1 root root  500056 Apr 27 02:03 ref.fa.sa
-rw-r--r-- 1 root root   56824 Apr 27 02:04 aln.sam
```

This is problematic because when you're back in the host environment, you can't modify these files. To circumvent this, create a user that matches the host user by passing three environmental variables from the host to the container.

``` bash
docker run -it \
-v ~/my_data:/data \
-e MYUID=`id -u` \
-e MYGID=`id -g` \
-e ME=`whoami` \
bwa /bin/bash
```

Use the steps below to create an identical user inside the container.

``` bash
adduser --quiet --home /home/san/$ME --no-create-home --gecos "" --shell /bin/bash --disabled-password $ME

# optional: give yourself admin privileges
echo "%$ME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# update the IDs to those passed into Docker via environment variable
sed -i -e "s/1000:1000/$MYUID:$MYGID/g" /etc/passwd
sed -i -e "s/$ME:x:1000/$ME:x:$MYGID/" /etc/group

# su - as the user
exec su - $ME

# run BWA again, after you have deleted the old files as root
bwa index ref.fa
bwa mem ref.fa l100_n100_d400_31_1.fq l100_n100_d400_31_2.fq > aln.sam

# check output
ls -lrt
total 2816
-rw-r--r-- 1 dtang dtang 1000015 Apr 27 02:00 ref.fa
-rw-r--r-- 1 dtang dtang   21478 Apr 27 02:00 l100_n100_d400_31_2.fq
-rw-r--r-- 1 dtang dtang   21478 Apr 27 02:00 l100_n100_d400_31_1.fq
-rw-r--r-- 1 dtang dtang     119 Apr 27 02:01 run.sh
-rw-rw-r-- 1 dtang dtang 1000072 Apr 27 02:12 ref.fa.bwt
-rw-rw-r-- 1 dtang dtang  250002 Apr 27 02:12 ref.fa.pac
-rw-rw-r-- 1 dtang dtang      40 Apr 27 02:12 ref.fa.ann
-rw-rw-r-- 1 dtang dtang      12 Apr 27 02:12 ref.fa.amb
-rw-rw-r-- 1 dtang dtang  500056 Apr 27 02:12 ref.fa.sa
-rw-rw-r-- 1 dtang dtang   56824 Apr 27 02:12 aln.sam

# exit container
exit
```

The files will be saved in `~/my_data` on the host.

``` bash
ls -lrt ~/my_data
total 2816
-rw-r--r-- 1 dtang dtang 1000015 Apr 27 10:00 ref.fa
-rw-r--r-- 1 dtang dtang   21478 Apr 27 10:00 l100_n100_d400_31_2.fq
-rw-r--r-- 1 dtang dtang   21478 Apr 27 10:00 l100_n100_d400_31_1.fq
-rw-r--r-- 1 dtang dtang     119 Apr 27 10:01 run.sh
-rw-rw-r-- 1 dtang dtang 1000072 Apr 27 10:12 ref.fa.bwt
-rw-rw-r-- 1 dtang dtang  250002 Apr 27 10:12 ref.fa.pac
-rw-rw-r-- 1 dtang dtang      40 Apr 27 10:12 ref.fa.ann
-rw-rw-r-- 1 dtang dtang      12 Apr 27 10:12 ref.fa.amb
-rw-rw-r-- 1 dtang dtang  500056 Apr 27 10:12 ref.fa.sa
-rw-rw-r-- 1 dtang dtang   56824 Apr 27 10:12 aln.sam
```

### File Permissions 2

An easier way to set file permissions is to use the `-u` parameter.

``` bash
# assuming blah.fa exists in /local/data/
docker run -v /local/data:/data -u `stat -c "%u:%g" /local/data` bwa bwa index /data/blah.fa
```

### Read only

To mount a volume but with read-only permissions, append `:ro` at the end.

``` bash
docker run --rm -v $(pwd):/work:ro davetang/bwa:0.7.17 touch test.txt
```

    ## touch: cannot touch 'test.txt': Read-only file system

## Removing the image

Use `docker rmi` to remove an image. You will need to remove any stopped containers first before you can remove an image. Use `docker ps -a` to find stopped containers and `docker rm` to remove these containers.

Let's pull the `busybox` image.

``` bash
docker pull busybox
```

    ## Using default tag: latest
    ## latest: Pulling from library/busybox
    ## b71f96345d44: Pulling fs layer
    ## b71f96345d44: Verifying Checksum
    ## b71f96345d44: Download complete
    ## b71f96345d44: Pull complete
    ## Digest: sha256:0f354ec1728d9ff32edcd7d1b8bbdfc798277ad36120dc3dc683be44524c8b60
    ## Status: Downloaded newer image for busybox:latest
    ## docker.io/library/busybox:latest

Check out `busybox`.

``` bash
docker images busybox
```

    ## REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
    ## busybox      latest    69593048aa3a   2 months ago   1.24MB

Remove `busybox`.

``` bash
docker rmi busybox
```

    ## Untagged: busybox:latest
    ## Untagged: busybox@sha256:0f354ec1728d9ff32edcd7d1b8bbdfc798277ad36120dc3dc683be44524c8b60
    ## Deleted: sha256:69593048aa3acfee0f75f20b77acb549de2472063053f6730c4091b53f2dfb02
    ## Deleted: sha256:5b8c72934dfc08c7d2bd707e93197550f06c0751023dabb3a045b723c5e7b373

## Committing changes

Generally, it is better to use a Dockerfile to manage your images in a documented and maintainable way but if you still want to [commit changes](https://docs.docker.com/engine/reference/commandline/commit/) to your container (like you would for Git), read on.

When you log out of a container, the changes made are still stored; type `docker ps -a` to see all containers and the latest changes. Use `docker commit` to commit your changes.

``` bash
docker ps -a

# git style commit
# -a, --author=       Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
# -m, --message=      Commit message
docker commit -m 'Made change to blah' -a 'Dave Tang' <CONTAINER ID> <image>

# use docker history <image> to check history
docker history <image>
```

## Access running container

To access a container that is already running, perhaps in the background (using detached mode: `docker run` with `-d`) use `docker ps` to find the name of the container and then use `docker exec`.

In the example below, my container name is `rstudio_dtang`.

``` bash
docker exec -it rstudio_dtang /bin/bash
```

## Cleaning up exited containers

I typically use the `--rm` flag with `docker run` so that containers are automatically removed after I exit them. However, if you don't use `--rm`, by default a container's file system persists even after the container exits. For example:

``` bash
docker run hello-world
```

    ## 
    ## Hello from Docker!
    ## This message shows that your installation appears to be working correctly.
    ## 
    ## To generate this message, Docker took the following steps:
    ##  1. The Docker client contacted the Docker daemon.
    ##  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    ##     (amd64)
    ##  3. The Docker daemon created a new container from that image which runs the
    ##     executable that produces the output you are currently reading.
    ##  4. The Docker daemon streamed that output to the Docker client, which sent it
    ##     to your terminal.
    ## 
    ## To try something more ambitious, you can run an Ubuntu container with:
    ##  $ docker run -it ubuntu bash
    ## 
    ## Share images, automate workflows, and more with a free Docker ID:
    ##  https://hub.docker.com/
    ## 
    ## For more examples and ideas, visit:
    ##  https://docs.docker.com/engine/userguide/

Show all containers.

``` bash
docker ps -a
```

    ## CONTAINER ID   IMAGE         COMMAND    CREATED        STATUS                              PORTS     NAMES
    ## eec0c54887dd   hello-world   "/hello"   1 second ago   Exited (0) Less than a second ago             bold_bhaskara

We can use a sub-shell to get all (`-a`) container IDs (`-q`) that have exited (`-f status=exited`) and then remove them (`docker rm -v`).

``` bash
docker rm -v $(docker ps -a -q -f status=exited)
```

    ## eec0c54887dd

Check to see if the container still exists.

``` bash
docker ps -a
```

    ## CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

We can set this up as a Bash script so that we can easily remove exited containers. In the Bash script `-z` returns true if `$exited` is empty, i.e. no exited containers, so we will only run the command when `$exited` is not true.

``` bash
cat clean_up_docker.sh
```

    ## #!/usr/bin/env bash
    ## 
    ## set -euo pipefail
    ## 
    ## exited=`docker ps -a -q -f status=exited`
    ## 
    ## if [[ ! -z ${exited} ]]; then
    ##    docker rm -v $(docker ps -a -q -f status=exited)
    ## fi
    ## 
    ## exit 0

As I have mentioned, you can use the [--rm](https://docs.docker.com/engine/reference/run/#clean-up---rm) parameter to automatically clean up the container and remove the file system when the container exits.

``` bash
docker run --rm hello-world
```

    ## 
    ## Hello from Docker!
    ## This message shows that your installation appears to be working correctly.
    ## 
    ## To generate this message, Docker took the following steps:
    ##  1. The Docker client contacted the Docker daemon.
    ##  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    ##     (amd64)
    ##  3. The Docker daemon created a new container from that image which runs the
    ##     executable that produces the output you are currently reading.
    ##  4. The Docker daemon streamed that output to the Docker client, which sent it
    ##     to your terminal.
    ## 
    ## To try something more ambitious, you can run an Ubuntu container with:
    ##  $ docker run -it ubuntu bash
    ## 
    ## Share images, automate workflows, and more with a free Docker ID:
    ##  https://hub.docker.com/
    ## 
    ## For more examples and ideas, visit:
    ##  https://docs.docker.com/engine/userguide/

No containers.

``` bash
docker ps -a
```

    ## CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

## Installing Perl modules

Use `cpanminus`.

``` bash
apt-get install -y cpanminus

# install some Perl modules
cpanm Archive::Extract Archive::Zip DBD::mysql
```

## Creating a data container

This [guide on working with Docker data volumes](https://www.digitalocean.com/community/tutorials/how-to-work-with-docker-data-volumes-on-ubuntu-14-04) provides a really nice introduction. Use `docker create` to create a data container; the `-v` indicates the directory for the data container; the `--name data_container` indicates the name of the data container; and `ubuntu` is the image to be used for the container.

``` bash
docker create -v /tmp --name data_container ubuntu
```

If we run a new Ubuntu container with the `--volumes-from` flag, output written to the `/tmp` directory will be saved to the `/tmp` directory of the `data_container` container.

``` bash
docker run -it --volumes-from data_container ubuntu /bin/bash
```

## R

Use images from [The Rocker Project](https://www.rocker-project.org/), for example `rocker/r-ver:4.1.0`.

``` bash
docker run --rm rocker/r-ver:4.1.0
```

    ## 
    ## R version 4.1.0 (2021-05-18) -- "Camp Pontanezen"
    ## Copyright (C) 2021 The R Foundation for Statistical Computing
    ## Platform: x86_64-pc-linux-gnu (64-bit)
    ## 
    ## R is free software and comes with ABSOLUTELY NO WARRANTY.
    ## You are welcome to redistribute it under certain conditions.
    ## Type 'license()' or 'licence()' for distribution details.
    ## 
    ## R is a collaborative project with many contributors.
    ## Type 'contributors()' for more information and
    ## 'citation()' on how to cite R or R packages in publications.
    ## 
    ## Type 'demo()' for some demos, 'help()' for on-line help, or
    ## 'help.start()' for an HTML browser interface to help.
    ## Type 'q()' to quit R.
    ## 
    ## >

## Saving and transferring a Docker image

You should just share the Dockerfile used to create your image but if you need another way to save and share an iamge, see [this post](http://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-via-repository) on Stack Overflow.

``` bash
docker save -o <save image to path> <image name>
docker load -i <path to image tar file>
```

Here's an example.

``` bash
# save on Unix server
docker save -o davebox.tar davebox

# copy file to MacBook Pro
scp davetang@192.168.0.31:/home/davetang/davebox.tar .

docker load -i davebox.tar 
93c22f563196: Loading layer [==================================================>] 134.6 MB/134.6 MB
...

docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
davebox             latest              d38f27446445        10 days ago         3.46 GB

docker run davebox samtools

Program: samtools (Tools for alignments in the SAM format)
Version: 1.3 (using htslib 1.3)

Usage:   samtools <command> [options]
...
```

## Pushing to Docker Hub

Create an account on [Docker Hub](https://hub.docker.com/); my account is `davetang`. Use `docker login` to login and use `docker push` to push to Docker Hub (run `docker tag` first if you didn't name your image in the format of `yourhubusername/newrepo`).

``` bash
docker login

# create repo on Docker Hub then tag your image
docker tag bb38976d03cf yourhubusername/newrepo

# push
docker push yourhubusername/newrepo
```

## Tips

Tip from <https://support.pawsey.org.au/documentation/display/US/Containers>: each RUN, COPY, and ADD command in a Dockerfile generates another layer in the container thus increasing its size; use multi-line commands and clean up package manager caches to minimise image size:

``` bash
RUN apt-get update \
      && apt-get install -y \
         autoconf \
         automake \
         gcc \
         g++ \
         python \
         python-dev \
      && apt-get clean all \
      && rm -rf /var/lib/apt/lists/*
```

## Useful links

-   [A quick introduction to Docker](http://blog.scottlowe.org/2014/03/11/a-quick-introduction-to-docker/)
-   [The BioDocker project](https://github.com/BioDocker/biodocker); check out their [Wiki](https://github.com/BioDocker/biodocker/wiki), which has a lot of useful information
-   [The impact of Docker containers on the performance of genomic pipelines](http://www.ncbi.nlm.nih.gov/pubmed/26421241)
-   [Learn enough Docker to be useful](https://towardsdatascience.com/learn-enough-docker-to-be-useful-b0b44222eef5)
-   [10 things to avoid in Docker containers](http://developers.redhat.com/blog/2016/02/24/10-things-to-avoid-in-docker-containers/)
-   The [Play with Docker classroom](https://training.play-with-docker.com/) brings you labs and tutorials that help you get hands-on experience using Docker
-   [Shifter](https://github.com/NERSC/shifter) enables container images for HPC
-   <http://biocworkshops2019.bioconductor.org.s3-website-us-east-1.amazonaws.com/page/BioconductorOnContainers__Bioconductor_Containers_Workshop/>
