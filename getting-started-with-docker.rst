.. Docker Training documentation master file, created by
   sphinx-quickstart on Fri Jun  7 12:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Getting Started with Docker
===========================

What is Docker?
***************
Simply put, Docker is a containerization technology that enables us to 
cleanly abstract an environment configuration and dependencies to a file 
(or set of files), and run it in a protected, isolated environment on a host.
Docker is similar to but more performant than, a virtual machine.\



What is containerization?
*************************
Containerization involves bundling an application together with all of its
related configuration files, libraries and dependencies required for it to run 
in an efficient and bug-free way across different computing environments.



How is this different from virtual machines?
********************************************
Virtual machines (VM) uses virtualization technology, whereby a hypervisor is
installed on the host which simulates another physical machine that can run 
another operating system on top of the host machine. Virtualization, like
Docker, offers isolation and flexibility of execution of software within a 
particular environment. However, virtualization has a relatively high cost 
of resource utilization and CPU usage. Also, each VM has its own copy of 
an Operating System, applications and their related files, libraries 
and dependencies.



Docker uses less space and memory than running software within different 
VMs since it does not require a separate copy of an Operating System to run
on the host machine. Docker uses the kernel of the host machine to create, 
build and run applications inside of a container.



What is a Dockerfile?
*********************
A Dockerfile is a file that contains a set of instructions that describe 
an environment configuration. For example, a React web app requires a NodeJS
environment that should be accessible to anyone on the internet, the
Dockerfile would look something like the following::

    FROM node:latest
    RUN apt-get install -y vim
    EXPOSE 80


What is a Docker Image?
***********************
A Docker image is a pre-built Dockerfile. It\’s ready to run and can be run
on any host that has Docker installed. The concept of images is where Docker
gets its portability. In our previous example, the latest version of the node
Docker image is being used in our Dockerfile.



What is a Docker Container?
***************************
A Docker container is an instance of a Docker image. Containers can be started,
running, restarted, and stopped. We\’re able to create as many containers from
a single image as we need. This concept helps to scale up a service easier.



The capabilities of building, managing and sharing container images have
helped make Docker the de-facto standard for deployment of scalable
applications in the cloud. It is supported by many cloud providers and
CI frameworks.



What are Docker Volumes?
************************
Volumes are stored in a part of the host filesystem which is managed by Docker.

Uses of Volumes:
++++++++++++++++


* decouple the container from the storage so that even when the container
  is not running or is deleted, files and folders that are in the volume is
  accessible via the host filesystem.
* Sharing data between containers



What is Docker Compose?
***********************
Docker Compose is a Docker tool for defining, connecting, managing and
running multi-container Docker application. It mainly used to build
microservices, clusters and layers.

Why use Docker?
===============
Docker enables you to rapidly deploy server environments
in\“containers.\”You might question why to use Docker rather
than VMware or Oracle\’s VirtualBox?



On the Linux kernel, Docker utilizes the virtualization technology,
but it does not create virtual machines. However, depending on 
your version of Windows and MacOS, you\’ll have to use Docker with a 
virtual machine.