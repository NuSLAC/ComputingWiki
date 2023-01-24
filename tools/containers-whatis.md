# Introduction
This wiki covers the basics of how to use a _singularity container_. What is a _container_? What is _singularity_? Those questions are where we start below :) This wiki page is meant to be for a quick start. A full documentation about `singularity` can be found [here](https://www.sylabs.io/docs/).

## What is a container?
*Containerization* is operating system-level virtualization so that software applications can run in <u>isolated user spaces called *containers*</u> in any cloud or non-cloud environment, regardless of type or vendor. Individually each container simulates a different software application and runs isolated processes by bundling related configuration files, libraries, and dependencies. But collectively multiple containers share a common OS Kernel (summarized from [Wikipedia](https://en.wikipedia.org/wiki/Containerization_(computing))).

**In short**: a container ships/provides a whole environment to execute a program, which is essentially a whole operating system (*except for* the kernel to execute your program, see below for this point).

## What is Singularity?
It is a particular container software technology started by a team of developers at the Lawrence Berkeley National Lab (LBNL). You can read the [wikipedia page](https://en.wikipedia.org/wiki/Singularity_(software)) about it. The most popular container software is called `Docker` and it is used widely in non-academic applications. `Singularity` has features that are more suited for ours and other areas of academic research. Many academic computing clusters has accommodated `Singularity` therefore. Although we don't dig any further here, you can find more discussions about key differences between `Docker` and `Singularity` in the section ["what's different from docker?"](tools/containers-whatis?id=what-is-different-from-docker).


## Not a virtual machine?
The key difference between a container and a _virtual machine_ (VM) is whether the environment shares the underlying OS kernel or not. While running a VM instance implies running an independent operating system's kernel per instance, a container shares the host platform's kernel. This reduces much of resource usage per container instance (and hence a burden on the host platform). The idea is that, "hey, to run your python script, I don't need to be running an additional OS kernel..." ;)

## What is different from Docker?
`Docker` is _the_ most popular container software out there. Singularity is similar to docker except it is a _rootless executable_. When you run a `Singularity` container, you can execute with your own user account and also use parts of the host's file system with access writes unchanged. In contrast, running a `Docker` container requires a root previledge. This removes extra burdens for administrators to manage environments for users to run a program (i.e. users getting root permission can interrupt with administrative tasks), and/or removes burdens from the users for handling root previledge (big power comes with big responsibility).  

To be more specific with examples, Docker is great for running daemon jobs like hosting a web server or a database service on a cloud server machine (like AWS, Azure, gcloud, etc.) which are better run by the system and not by individual's "user account". Singularity is suited for running your personal code (such as analysis python script for your research) on a high performance computing (HPC) cluster. The key difference is in the needs for a run-environment. In the former, you typically want to avoid using a user account. In the latter, it is natural to use his/her user account as is for scientists. You don't want to have a root permission and by mistake remove someone else's data file, or even interrupt with the system.

Hopefully this made it clear that Singularity is not in competition against Docker. In fact it works nicely with docker: **you can build a singularity container based on a docker container.** So, as a singularity user, you can take benefits from both type of container images, plus HPC admins would love you. 

Finally, a _rootless container_ is one of major on-going discussion and development effort in their field. Singularity needs root access when you are building an image, so not a truly rootless container technology (but it is rootless at run time). You can google more about rootless container if you are interested int.






















