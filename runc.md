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
