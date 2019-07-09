# Using RunC to start a Container

By the end of this exercise, you should be able to:

 - Start and manage a container without docker
 - Explore the use of namespaces to isolate processes.
 
Relevance:

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

7.Launch a new container, this time into the background:

```bash
# runc run -d c1
# runc list
ID PID STATUS BUNDLE CREATED OWNER
c1 15231 running /root/container 2019-01-24T18:54:36.270242388Z root

```

8.Pause and resume the process in the container:

```bash
# runc pause c1
# runc state c1
{
"ociVersion": "0.6.0-dev",
"id": "c1",
"pid": 15058,
"bundlePath": "/root/container",
"rootfsPath": "/root/container/rootfs",
"status": "paused",
"created": "2016-12-23T04:01:50.624301713Z"
}
# runc resume c1
```

9. Find the PID number for the process:

```bash
# pgrep -f "sleep 1d"
12352
```

Take note of the PID number for the sleep process from your output:
Result: _______________________

10. Examine the namespaces in effect for the containerized process, and your current
shell:

```bash
# readlink /proc/PID_of_shell_from_step_9/ns/*
ipc:[4026532311]
mnt:[4026532309]
net:[4026532314]
pid:[4026532312]
user:[4026531837]
uts:[4026532310]
# readlink /proc/$$/ns/*
ipc:[4026531839]
mnt:[4026531840]
net:[4026531956]
pid:[4026531836]
user:[4026531837]
uts:[4026531838]
```

Note which namespaces are different (ipc, mnt, net, pid, and uts) and which are
the same (user).


11.Generate a report of how many namespaces are currently in use by processes
and how many processes are using each namespace:

```bash
# lsns
NS TYPE NPROCS PID USER COMMAND
. . . snip . . .
4026532298 mnt        1  2096 root sleep 1d
4026532299 uts        1  2096 root sleep 1d
4026532300 ipc        1  2096 root sleep 1d
. . . snip . . .
```

12. Remove the container and its related filesystem:

```bash
# runc kill c1 KILL
# runc list
ID PID STATUS BUNDLE CREATED
c1 2096 destroyed /root/container 2019-12-23T04:17:51.610890333Z
# runc delete c1
# runc list
ID PID STATUS BUNDLE CREATED
# cd
# rm -rf container/
```

13. Administrative privileges are no longer required; exit the root shell to return to an
unprivileged account.
