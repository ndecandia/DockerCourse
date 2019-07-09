# Installing Docker Engine

By the end of this exercise, you should be able to:

 - Install docker engine on linux
 - Check status and validate installation
 
 1. Install required packages. yum-utils provides the yum-config-manager utility, and device-mapper-persistent-data and lvm2 are required by the devicemapper storage driver.

```bash
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  ```
  
  2. Use the following command to set up the stable repository.
  
  ```bash
  $ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
  ```
  
  
  3. Install the latest version of Docker CE and containerd, or go to the next step to install a specific version:
  
  ```bash
  $ sudo yum install docker-ce docker-ce-cli containerd.io
  ```
  
  Docker is installed but not started. The docker group is created, but no users are added to the group.
  
  4. Start Docker.
  
  ```bash
 $ sudo systemctl enable --now docker
  ```
  
  5.Verify that Docker CE is installed correctly by running the hello-world image.
  
  ```bash
 $ sudo docker container run hello-world
  ```
  
  6. Check and validate your installation with the following commands. Examine the output.
  
  ```bash
  $ sudo docker version
  $ sudo docker info
  ```
  
