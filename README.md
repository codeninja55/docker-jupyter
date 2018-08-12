# Docker Jupyter
A Docker container for Jupyter notebook with multiple language kernels (Python3, C, C++, R, Keras and TensorFlow).

## Why?

* Containerised Jupyter Notebook environment allows portability of packages across operating systems.
* Simple installation via pre-built library images and `Dockerfile` directives.
* Fast installation of multiple kernels and packages at once to make the most of Jupyter Notebook's versatility.
* Separation of environment variables dedicated to Jupyter versus changing those of the OS. 

## Dependencies

* Docker >= 18.06.0-ce-win72 || 17.12.0-ce (Parrot OS Debian Stretch)
* Conda >= 4.5.9
* tini >= 0.18.0
* Jupyter
  * notebook >= 5.6.*
  * jupyterhub >= 0.9.*
  * jupyterlab >= 0.33.*
* tensorflow => 1.5
* keras >= 2.1

## What is Docker?

REF: https://docs.docker.com/get-started/

> Docker is a platform for developers and sysadmins to **develop, deploy, and run** applications with containers. The use of Linux containers to deploy applications is called *containerization*. Containers are not new, but their use for easily deploying applications is.
>
> Containerization is increasingly popular because containers are:
>
> - Flexible: Even the most complex applications can be containerized.
> - Lightweight: Containers leverage and share the host kernel.
> - Interchangeable: You can deploy updates and upgrades on-the-fly.
> - Portable: You can build locally, deploy to the cloud, and run anywhere.
> - Scalable: You can increase and automatically distribute container replicas.
> - Stackable: You can stack services vertically and on-the-fly.

### Images and Containers

> Images and containers
> A container is launched by running an image. An image is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.
>
> A container is a runtime instance of an image--what the image becomes in memory when executed (that is, an image with state, or a user process). You can see a list of your running containers with the command, docker ps, just as you would in Linux.

### Containers and virtual machines

> A container runs natively on Linux and shares the kernel of the host machine with other containers. It runs a discrete process, taking no more memory than any other executable, making it lightweight.
>
> By contrast, a virtual machine (VM) runs a full-blown “guest” operating system with virtual access to host resources through a hypervisor. In general, VMs provide an environment with more resources than most applications need.

## What is Jupyter?

REF: https://jupyter.org/index.html

> The Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling, data visualization, machine learning, and much more.
>
> **Features**
>
> * Language of choice: The Notebook has support for over 40 programming languages, including Python, R, Julia, and Scala.
> * Share notebooks: Notebooks can be shared with others using email, Dropbox, GitHub and the Jupyter Notebook Viewer.
> * Interactive output: Your code can produce rich, interactive output: HTML, images, videos, LaTeX, and custom MIME types. 
> * Big data integration: Leverage big data tools, such as Apache Spark, from Python, R and Scala. Explore that same data with pandas, scikit-learn, ggplot2, TensorFlow. 

## (Windows) Installation for Docker:

[Docker Install](https://docs.docker.com/docker-for-windows/install/)

### Test Docker version

Run `docker --version` and ensure that you have a supported version of Docker: 

```shell
> docker --version
Docker version 18.06.0-ce, build 0ffa825
```

Run `docker info` or (`docker version` without `--`) to view even more details about your docker installation: 

```shell
> docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.06.0-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
...
```

### Cheat Sheet

```shell
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

## (Parrot OS) Installation for Docker:

### Uninstall any old versions

```bash
$ sudo apt-get remove docker docker-engine docker.io
```

 ### Install Docker CE

```bash
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl \ 
	gnupg2 software-properties-common
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

$ sudo echo 'deb [arch=amd64] https://download.docker.com/linux/debian stretch stable' > /etc/apt/sources.list.d/docker.list
$ sudo apt update
$ sudo apt install docker-ce
$ docker --version
Docker version 17.12.0-ce, build c97c6d6
```

### Manage Docker as a non-root user

### Create the `docker` group and add your user.

```bash
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

## The Jupyter Stacks Docker Image

![Jupyter Stacks](./jupyter-stacks.svg)

## Simple Installation from Git

```bash
$ git clone https://github.com/codeninja55/docker-jupyter.git
$ cd jupyter-image
$ docker build -t jupyter-image .
$ docker run -p 8888:8888 jupyter-image
```

