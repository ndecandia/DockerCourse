# Docker Registry

By the end of this exercise, you should be able to:

 - Create a docker registry
 - Explore registry and push images


1. Start a registry with the default configuration included within the image:

```bash
$ docker container run -dp 5000:5000 --name reg registry:2.6
Unable to find image 'registry:2.6' locally
2.5: Pulling from registry
3690ec4760f9: Pull complete
930045f1e8fb: Pull complete
feeaa90cbdbc: Pull complete
61f85310d350: Pull complete
b6082c239858: Pull complete
Digest: sha256:b43824c6e32b1ffc6a7b362ad18d07e72a3def816005fa40a1931958b65f1b5e
Status: Downloaded newer image for registry:2.6
484e480846d3e73b3a8301a095a9d1227359fb491b4c4f55684dbf36df84ee65
```

2. Verify that it reports in as a registry:

```bash
$ curl -i 127.0.0.1:5000/v2/
HTTP/1.1 200 OK
Content-Length: 2
Content-Type: application/json; charset=utf-8
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
Date: Thu, 03 Nov 2016 01:24:40 GMT
```

3. Look at what default command and entrypoint is defined by the image:

```bash
$ docker container inspect reg | egrep -A1 "(Cmd|Entry)"
"Cmd": [
"/etc/docker/registry/config.yml"
--
"Entrypoint": [
"/entrypoint.sh"
```

4. Enter the registry container and look around to better understand what it contains
and how it operates:

```bash
$ docker container exec -ti reg /bin/sh
/ # cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.4.6
PRETTY_NAME="Alpine Linux v3.4"
HOME_URL="http://alpinelinux.org"
BUG_REPORT_URL="http://bugs.alpinelinux.org"
/ # ps aux
PID USER TIME COMMAND
1 root 0:00 registry serve /etc/docker/registry/config.yml
10 root 0:00 /bin/sh
18 root 0:00 ps aux
/ # ls -l
total 24
drwxr-xr-x 2 root root 4096 Oct 19 00:10 bin
drwxr-xr-x 5 root root 360 Nov 3 01:21 dev
-rwxrwxr-x 1 root root 155 Oct 19 00:10 entrypoint.sh
drwxr-xr-x 17 root root 4096 Nov 3 01:21 etc
drwxr-xr-x 2 root root 6 Oct 18 18:58 home
drwxr-xr-x 5 root root 4096 Oct 19 00:10 lib
lrwxrwxrwx 1 root root 12 Oct 18 18:58 linuxrc -> /bin/busybox
. . . snip . . .
/ # cat entrypoint.sh
#!/bin/sh
set -e
case "$1" in
*.yaml|*.yml) set -- registry serve "$@" ;;
serve|garbage-collect|help|-*) set -- registry "$@" ;;
esac
exec "$@"
/ # exit

```

5. Copy the config out of the container so that it can be examined and changed:

```bash
$ mkdir /labfiles/docker/reg_config
$ cd /labfiles/docker/reg_config
$ docker container cp reg:/etc/docker/registry/config.yml .
$ cat config.yml
version: 0.1
log:
fields:
service: registry
storage:
cache:
blobdescriptor: inmemory
filesystem:
rootdirectory: /var/lib/registry
http:
addr: :5000
headers:
X-Content-Type-Options: [nosniff]
health:
storagedriver:
enabled: true
interval: 10s
threshold: 3
```

6. Modify the config so the registry runs on port 80:

```bash
$ sed -i 's/5000/80/' config.yml
```

7. Run a new registry with the modified config:

```bash
$ docker container rm -fv reg
$ docker image tag registry:2.6 registry
$ docker container run -dp 80:80 --name reg -v $(pwd)/config.yml:/config.yml registry config.yml
fd9a3267bbfe963d34aa60ef05558924eaa5a08b594a1c17b9bbbc7ba10d9e1c
$ curl -i 127.0.0.1/v2/
HTTP/1.1 200 OK
Content-Length: 2
Content-Type: application/json; charset=utf-8
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
Date: Thu, 03 Nov 2016 01:52:07 GMT
```

8. Tag an image with your local registry information and push it to the new registry:

```bash
$ docker image pull busybox:1.30.1
$ docker image tag busybox:1.30.1 localhost/busybox
$ docker image push localhost/busybox
The push refers to a repository [localhost/busybox]
5f70bf18a086: Pushed
1834950e52ce: Pushed
latest: digest: sha256:9e19eb0c215b760dc7f268402ffe83cb0f063bbcbb181bc25d2d75a9531ea117 size: 733
```

9. Identify what internal Docker volume is storing the registry's images and examine
it:

```bash
$ docker container inspect -f '{{json .Mounts }}' reg | tr , '\n'
[{"Source":"/home/guru/reg_config/config.yml"
"Destination":"/config.yml"
"Mode":""
"RW":true
"Propagation":"rprivate"}
{"Name":"6fd6db41df7a354a8a6f20386326ae1129a18f5f6bb51b3d7b07bdcbc4cebc26"
"Source":"/var/lib/docker/volumes/6fd6db41df7a354a8a6f20386326ae1129a18f5f6bb51b3d7b07bdcbc4cebc26/_data"
"Destination":"/var/lib/registry"
"Driver":"local"
"Mode":""
"RW":true
"Propagation":""}]
$ docker volume ls
DRIVER VOLUME NAME
local 6fd6db41df7a354a8a6f20386326ae1129a18f5f6bb51b3d7b07bdcbc4cebc26
$ docker container exec reg find /var/lib/registry -type f
/var/lib/registry/docker/registry/v2/repositories/busybox/_layers/sha256/a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4//var/lib/registry/docker/registry/v2/repositories/busybox/_layers/sha256/385e281300cc6d88bdd155e0931fbdfbb1801c2b0265340a40481ee2b733ae66/
. . . snip . . .
```

10. Push another tag for an image and use the registry API to list all tags:

```bash
$ curl localhost/v2/_catalog
{"repositories":["busybox"]}
$ docker image tag localhost/busybox{,:v2}
$ docker image push localhost/busybox:v2
The push refers to a repository [localhost/busybox]
5f70bf18a086: Layer already exists
1834950e52ce: Layer already exists
v2: digest: sha256:9e19eb0c215b760dc7f268402ffe83cb0f063bbcbb181bc25d2d75a9531ea117 size: 733
$ curl localhost/v2/busybox/tags/list
{"name":"busybox","tags":["latest","v2"]}
```

