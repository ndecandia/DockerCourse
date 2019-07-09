# Create Images using Dockerfiles (3/3) | Multi-Stage

By the end of this exercise, you should be able to:

- Write a Dockerfile that describes multiple images, which can copy files from one image to the next.


## Defining a multi-stage build

1.  Make a fresh folder `~/multi` to do this exercise in, and `cd` into it.

2.  Add a file `hello.c` to the `multi` folder containing **Hello World** in C:

    ```c
    #include <stdio.h>

    int main (void)
    {
        printf ("Hello, world!\n");
        return 0;
    }
    ```

3.  Try compiling and running this right on the host OS:

    ```bash
    [centos@desotech multi]$ gcc -Wall hello.c -o hello
    [centos@desotech multi]$ ./hello
    ```

4.  Now let's Dockerize our hello world application. Add a `Dockerfile` to the `multi` folder with this content:

    ```bash
    FROM alpine:3.5
    RUN apk update && \
        apk add --update alpine-sdk
    RUN mkdir /app
    WORKDIR /app
    COPY hello.c /app
    RUN mkdir bin
    RUN gcc -Wall hello.c -o bin/hello
    CMD /app/bin/hello
    ```

5.  Build the image and observe its size:

    ```bash
    [centos@desotech multi]$ docker image build -t my-app-large .
    [centos@desotech multi]$ docker image ls | grep my-app-large

    REPOSITORY     TAG      IMAGE ID       CREATED         SIZE
    my-app-large   latest   a7d0c6fe0849   3 seconds ago   189MB
    ```

6.  Test the image to confirm it actually works:

    ```bash
    [centos@desotech multi]$ docker container run my-app-large
    ```

    It should print "hello world" in the console.

7.  Update your Dockerfile to use an `AS` clause on the first line, and add a second stanza describing a second build stage:

    ```bash
    FROM alpine:3.5 AS build
    RUN apk update && \
        apk add --update alpine-sdk
    RUN mkdir /app
    WORKDIR /app
    COPY hello.c /app
    RUN mkdir bin
    RUN gcc -Wall hello.c -o bin/hello

    FROM alpine:3.5
    COPY --from=build /app/bin/hello /app/hello
    CMD /app/hello
    ```

8.  Build the image again and compare the size with the previous version:

    ```bash
    [centos@desotech multi]$ docker image build -t my-app-small .
    [centos@desotech multi]$ docker image ls | grep 'my-app-'

    REPOSITORY     TAG      IMAGE ID       CREATED              SIZE
    my-app-small   latest   f49ec3971aa6   6 seconds ago        4.01MB
    my-app-large   latest   a7d0c6fe0849   About a minute ago   189MB
    ```

    As expected, the size of the multi-stage build is much smaller than the large one since it does not contain the Alpine SDK.

9.  Finally, make sure the app actually works:

    ```bash
    [centos@desotech multi]$ docker container run --rm my-app-small
    ```

    You should get the expected 'Hello, World!' output from the container with just the required executable.

## Building Intermediate Images

In the previous step, we took our compiled executable from the first build stage, but that image wasn't tagged as a regular image we can use to start containers with; only the final `FROM` statement generated a tagged image. In this step, we'll see how to persist whichever build stage we like.

1.  Build an image from the `build` stage in your Dockerfile using the `--target` flag:

    ```bash
    [centos@desotech multi]$ docker image build -t my-build-stage --target build .
    ```

    Notice all its layers are pulled from the cache; even though the build stage wasn't tagged originally, its layers are nevertheless persisted in the cache.

2.  Run a container from this image and make sure it yields the expected result:

    ```bash
    [centos@desotech multi]$ docker container run -it --rm my-build-stage /app/bin/hello
    ```

3.  List your images again to see the size of `my-build-stage` compared to the small version of the app.

## Optional: Building from Scratch

So far, every image we've built has been based on a pre-existing image, referenced in the `FROM` command. But what if we want to start from nothing, and build a completely original image? For this, we can build `FROM scratch`.

1.  In a fresh directoy `~/scratch`, create a file `sleep.c` that just launches a sleeping process for an hour:

    ```c
    #include <stdio.h>
    #include <unistd.h>
    int main()
    {
      int delay = 3600; //sleep for 1 hour
      printf ("Sleeping for %d second(s)...\n", delay);
      sleep(delay);
      return 0;
    }
    ```

2.  Create a file `Dockerfile` to build this sleep program in a build stage, and then copy it to a `scratch`-based image:

    ```
    FROM alpine:3.8 AS build
    RUN ["apk", "update"]
    RUN ["apk", "add", "--update", "alpine-sdk"]
    COPY sleep.c /
    RUN ["gcc", "-static", "sleep.c", "-o", "sleep"]

    FROM scratch
    COPY --from=build /sleep /sleep
    CMD ["/sleep"]
    ```

    This image will contain nothing but our executable and the bare minimum file structure Docker needs to stand up a container filesystem. Note we're statically linking the `sleep.c` binary, so it will have everything it needs bundled along with it, not relying on the rest of the container's filesystem for anything.

3.  Build your image:

    ```bash
    [centos@desotech scratch]$ docker image build -t sleep:scratch .
    ```

4.  List your images, and search for the one you just built:

    ```bash
    [centos@desotech scratch]$ docker image ls | grep scratch

    REPOSITORY  TAG       IMAGE ID       CREATED         SIZE
    sleep       scratch   1b68b20a85a8   9 minutes ago   128kB
    ```

    This image is only 128 kB, as tiny as possible.

5.  Run your image, and check out its filesystem; we can't list directly inside the container, since `ls` isn't installed in this ultra-minimal image, so we have to find where this container's filesystem is mounted on the host. Start by finding the PID of your sleep process after its running:

    ```bash
    [centos@desotech scratch]$ docker container run --name sleeper -d sleep:scratch
    [centos@desotech scratch]$ docker container top sleeper

    UID   PID   PPID  C  STIME  TTY  TIME     CMD
    root  1190  1174  0  15:21  ?    00:00:00 /sleep
    ```

    In this example, the PID for `sleep` is 1190.

6.  List your container's filesystem from the host using this PID:

    ```bash
    [centos@desotech scratch]$ sudo ls /proc/<PID>/root

    dev  etc  proc  sleep  sys
    ```

    We see not only our binary `sleep` but a bunch of other folders and files. Where does these come from? runC, the tool for spawning and running containers, requires a json config of the container and a root file system. At runtime, Docker Engine adds these minimum requirements to form the most minimal container filesystem possible.

7.  Clean up by deleting your container:

    ```bash
    [centos@desotech scratch]$ docker container rm -f sleeper
    ```


## Conclusion

In this exercise, you created a Dockerfile defining multiple build stages. Being able to take artifacts like compiled binaries from one image and insert them into another allows you to create very lightweight images that do not include developer tools or other unnecessary components in your production-ready images, just like how you currently probably have separate build and run environments for your software. This will result in containers that start faster, and are less vulnerable to attack.
