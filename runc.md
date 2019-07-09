# Using RunC to start a Container

By the end of this exercise, you should be able to:

 - Start and manage a container without docker
 - Explore the use of namespaces to isolate processes.
 
Relevance
runC provides a simple OCI compliant, libcontainer based command for
creating containers. This allows for exploring the basic concepts of
containers without involving Docker.


1. The following actions require administrative privileges. Switch to a root login.

2. Install runC and its dependencies on your first assigned lab station (node1):

```bash
[node1]# yum install -y containerd.io libseccomp
. . . snip . . .
Running transaction
Installing : containerd.io-1.2.4-3.1.el7.x86_64 1/2
Installing : libseccomp-2.3.1-2.el7.x86_64 2/2
Verifying : libseccomp-2.3.1-2.el7.x86_64 1/2
Verifying : containerd.io-1.2.4-3.1.el7.x86_64 2/2
Installed:
containerd.io.x86_64 0:1.2.4-3.1.el7 libseccomp.x86_64 0:2.3.1-2.el7
Complete!

```

3.Create a simple root filesystem for the container to use:

```bash
# mkdir -p /root/container/rootfs/
# cd $_
# tar xf /labfiles/docker/alpine.tar
# ls
bin dev etc home lib linuxrc media mnt proc root run sbin srv sys tmp usr var

```

4.Create a runC specification file and examine it:

```bash
# cd ..
# runc spec
# less config.json
```

When done, press 'q' to quit

5.Launch a shell running in the container and explore the isolating effect of
namespaces:

```bash
# runc run c1
/ # pwd
/
/ # ls
bin etc lib media proc run srv tmp var
dev home linuxrc mnt root sbin sys usr
/ # ls /dev/
console fd mqueue ptmx random stderr stdout urandom
core full null pts shm stdin tty zero
/ # ps -ef
PID USER TIME COMMAND
1 root 0:00 sh
7 root 0:00 ps -ef
/ # hostname
/ # ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
/ # exit
```

Exiting the shell terminates the container and its associated namespaces:

```bash
# runc list
ID PID STATUS BUNDLE CREATE
```

6.Edit the config.json and change the default process that is started:

File: /root/container/config.json

   "process": {
              "terminal": true false,
              "user": {},
              "args": [
                      "sleep", "1d"
               ],

