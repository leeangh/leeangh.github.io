---
layout: post
title:  "Play with LXC"
tags: [Container, LXC]
---

### [LXC](https://linuxcontainers.org/lxc/introduction/)

> LXC is a userspace interface for the Linux kernel containment features. Through a powerful API and simple tools, it lets Linux users easily create and manage system or application containers.
> >LXC containers are often considered as something in the middle between a chroot and a full fledged virtual machine. The goal of LXC is to create an environment as close as possible to a standard Linux installation but without the need for a separate kernel.

I have been studying Docker recently. While looking at it's history, I just learned that the Docker was started based on LXC and just know that there are many other container technologies.

In my opinion, try to have a basic knowledge of how LXC or other container works would help me more deeply understanding on Docker. Well, at least a complementary.

I'm running a Manjaro Linux.

## 1. Install LXC
```bash
yaourt -S lxc
```
After this package is isntalled, you should see following commands are avaiable on you computer:
```
lxc-checkconfig    lxc-copy           lxc-execute        lxc-monitor        lxc-to-lxd         lxc-update-config
lxc-attach         lxc-checkpoint     lxc-create         lxc-freeze         lxc-snapshot       lxc-top            lxc-usernsexec   
lxc-autostart      lxc-config         lxc-destroy        lxc-info           lxc-start          lxc-unfreeze       lxc-wait         
lxc-cgroup         lxc-console        lxc-device         lxc-ls             lxc-stop           lxc-unshare
```

**Note!** If you have the [LXD](https://linuxcontainers.org/lxd/introduction/) installed, you will see a command `lxc`. But, it's not for LXC, it is a client command line for LXD.

### 1.1 Package for Bridge Network

```bash
yaourt -S bridge-utils
```

It provides utilities for configuring the Linux ethernet bridge, will be used for configure bridge network interface for LXC container's network setup.

## 2. More Configuration Check for LXC

After LXC is installed, we still needed to run a check command to make sure all prerequisties are ready for LXC.

A 'cgroups' mount point is mandantory for LXC.

```bash
⇒  lxc-checkconfig
--- Namespaces ---
Namespaces: enabled
Utsname namespace: enabled
Ipc namespace: enabled
Pid namespace: enabled
User namespace: enabled
Network namespace: enabled

--- Control groups ---
Cgroups: enabled

Cgroup v1 mount points: 
/sys/fs/cgroup/systemd
..........

Cgroup v2 mount points: 
/sys/fs/cgroup/unified

Cgroup v1 clone_children flag: enabled
Cgroup device: enabled
Cgroup sched: enabled
Cgroup cpu account: enabled
Cgroup memory controller: enabled
Cgroup cpuset: enabled

...................
```
[More](./lxc_checkconfig)

## 3. Create Your First Container

### 3.1 Create the Container Image
```bash
arthur@arthur-pc:~|⇒  sudo lxc-create -t download -n debian_container
Setting up the GPG keyring
Downloading the image index

---
DIST    RELEASE ARCH    VARIANT BUILD
---
alpine  3.4     amd64   default 20180627_17:50
.....
debian  wheezy  amd64   default 20180627_05:24
....
ubuntu  xenial  amd64   default 20180920_07:43
...
---

Distribution: 
debian
Release: 
wheezy
Architecture: 
amd64

Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs

---
You just created an Debian wheezy amd64 (20180627_05:24) container.

To enable SSH, run: apt install openssh-server
No default root or user password are set by LXC.
```

### 3.2 Check & Launch the Container
```bash
$ sudo lxc-ls --fancy       
NAME                 STATE   AUTOSTART GROUPS IPV4 IPV6 UNPRIVILEGED 
debian_container     STOPPED 0         -      -    -    false

$ sudo lxc-start -n debian_container
```

Now we shall have the OS container running on the host:
```bash
$ sudo lxc-info -n debian_container
Name:           debian_container
State:          RUNNING
PID:            7159
IP:             192.168.122.149
CPU use:        1.04 seconds
BlkIO use:      9.63 MiB
Memory use:     18.77 MiB
KMem use:       5.81 MiB
Link:           veth9SIH8W
 TX bytes:      1.75 KiB
 RX bytes:      6.91 KiB
 Total bytes:   8.66 KiB
```

## 4. Step into the Container

Until now, we have a first OS container running, then we will try to step into the OS(Container).
```bash
$ sudo lxc-attach -n debian_container
root@debian_container:/# 
```

In this example, you would have noticed that the LXC container act as an OS vitualiztion tool. That's why we say, LXC can be used as an OS container or an application container. Here I only share with you very basic information of playing with LXC, more fun needs yourself to find out...