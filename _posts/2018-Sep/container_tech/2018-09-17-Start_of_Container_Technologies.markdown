---
layout: post
title:  "Start of Container Technologies"
tags: [Container, LXC, LXD, Docker, Kubernetes, Swarm, Mesos]
---

No matter which area of IT industry you have been working on, more or less you must have heard about the Container Technology. When talking about the Container, we always compare it with tranditional VM. But not necessary to argue about which is better, in my opinion they both are great technologies in their suitable areas.

But I'm not trying to write a complete manual for this huge topic, you can find so many great books or articles with fantastic content to help you to understand more details of `Container` or `Virtual Machine`. I was just want to share my roadmap on learnning this great technology, so if you found any thing wrong please correct me, and forgive me :blush:

## Container, Virtualization Technology

In simple terms, the main difference with Container and VM are:
* Where the virtualizaiton happens
* How they access the resources

> Container runs on top of host OS, shares the host OS kernel, resources(e.g. binaries, libraries). Containers were process-level isolated, less resources costed.

> Virtual Machine runs on top of a `Hypervisor`, as an independent OS. The hypervisor simulate all needed resources for running the Guest OSs.

**e.g. VM vs Docker:**

![VM-vs-Docker](/img/container/docker_vs_vmware.jpg)

The above picture doesn't represent the whole of `VM vs Container` and `Linux Container` contains much more than `Docker`. But why I directly use the piture of `VM vs Docker` because docker is the most widely used Container technology in the real world.

The Linux Container has two different types:
* Operation System Container (e.g. LXC)
* Application Container

**e.g. Docker vs LXC**

![lxc-vs-docker](/img/container/lxc-vs-docker.jpg)

## More, LXC? LXD, Docker?

What is LXC? LXD? Docker?

Container is not new, it has been introduced into Linux in the form of LXC over 10 Years and not only on Linux; BSD, Solaris, Windows have their own container implementations.

![normal-container-technologies](/img/container/container_technologies.png)

### [LXC](https://linuxcontainers.org/lxc/introduction/)

> LXC is a userspace interface for the Linux kernel containment features. Through a powerful API and simple tools, it lets Linux users easily create and manage system or application containers.
> >LXC containers are often considered as something in the middle between a chroot and a full fledged virtual machine. The goal of LXC is to create an environment as close as possible to a standard Linux installation but without the need for a separate kernel.

### [LXD](https://linuxcontainers.org/lxd/introduction/)

> LXD is a next generation system container manager. It offers a user experience similar to virtual machines but using Linux containers instead.
> LXD is built on top of LXC with new, better user expericence, it uses LXC through liblxc and its Go binding to create and manage the containers.

### [Docker](https://www.docker.com)

Docker has many same fundamentals as LXC but with much more features and better ecosytem, except that the docker is an application container while the LXC can be a OS container.

The very earlier version of Docker was based on LXC to provide single-application based, layered container, after then since version 0.9 Docker has it's own library - libcontainer. [`[DOCKER 0.9: INTRODUCING EXECUTION DRIVERS AND LIBCONTAINER]`](https://blog.docker.com/2014/03/docker-0-9-introducing-execution-drivers-and-libcontainer/)

## Cluster Management Tools

Indeed, containers are providing excellent application user experience. But along with the rapidly growth of the scale of the software deployment, container's creation, deployment and manipulation become to extreme complicate. Then, a cluster managment and orchestration too is needed.

There are three mostly known technologies:
* Kubernetes – Google
* Swarm – Docker
* Mesos – Apache

## Is VM dead?

Even containers has been widely used in almost everywhere, I believe it will never replace the VM. Looks like they have some common targets, but they are totally different technologies.

In a foreseeable future, VM will be reduced to smaller areas but it is still a complementary to docker, vice versa.