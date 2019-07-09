# Administer Containers (2)

By the end of this exercise, you should be able to:

- Run a container detached from the terminal
- Fetch the logs of a container
- Attach a terminal to the STDOUT of a running container

## Running a Container in the Background

1.  First try running a container as usual; the STDOUT and STDERR streams from whatever is PID 1 inside the container are directed to the terminal:

    ```bash
    [centos@desotech ~]$ docker container run centos:7 ping 127.0.0.1 -c 2

    PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
    64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.021 ms
    64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.029 ms

    --- 127.0.0.1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1019ms
    rtt min/avg/max/mdev = 0.021/0.025/0.029/0.004 ms
    ```

2.  The same process can be run in the background with the `-d` flag:

    ```bash
    [centos@desotech ~]$ docker container run -d centos:7 ping 127.0.0.1

    d5ef517cc113f36738005295066b271ae604e9552ce4070caffbacdc3893ae04
    ```

    This time, we only see the container's ID; its STDOUT isn't being sent to the terminal.

3.  Use this second container's ID to inspect the logs it generated:

    ```bash
    [centos@desotech ~]$ docker container logs <container ID>
    ```

    These logs correspond to STDOUT and STDERR from the container's PID 1. Also note when using container IDs: you don't need to specify the entire ID. Just enough characters from the start of the ID to uniquely identify it, often just 2 or 3, is sufficient.

## Attaching to Container Output

1.  We can attach a terminal to a container's PID 1 output with the `attach` command; try it with the last container you made in the previous step:

    ```bash
    [centos@desotech ~]$ docker container attach <container ID>
    ```

2.  We can leave attached mode by then pressing `CTRL+C`. After doing so, list your running containers; you should see that the container you attached to has been killed, since the `CTRL+C` issued killed PID 1 in the container, and therefore the container itself.

3.  Try running the same thing in detached interactive mode:

    ```bash
    [centos@desotech ~]$ docker container run -d -it centos:7 ping 127.0.0.1
    ```

4.  Attach to this container like you did the first one, but this time detach with `CTRL+P CTRL+Q` (sequential, not simultaneous), and list your running containers. In this case, the container should still be happily running in the background after detaching from it.

## Using Logging Options

1.  We saw previously how to read the entire log of a container's PID 1; we can also use a couple of flags to control what logs are displayed. `--tail n` limits the display to the last n lines; try it with the container that should be running from the last step:

    ```bash
    [centos@desotech ~]$ docker container logs --tail 5 <container ID>
    ```

    You should see the last 5 pings from this container.

2.  We can also follow the logs as they are generated with `-f`:

    ```bash
    [centos@desotech ~]$ docker container logs -f <container ID>
    ```

    The container's logs get piped in real time to the terminal (`CTRL+C` to break out of following mode - note this doesn't kill the process like when we attached to it, since now we're tailing the logs, not attaching to the process).

3.  Finally, try combining the tail and follow flags to begin following the logs from 10 lines back in history.

## Conclusion

In this exercise, we saw our first detached containers. Almost all containers you ever run will be running in detached mode; you can use `container attach` to interact with their PID 1 processes, as well as `container logs` to fetch their logs. Note that both `attach` and `logs` interact with the PID 1 process only - if you launch child processes inside a container, it's up to you to manage their STDOUT and STDERR streams. Also, be careful when killing processes after attaching to a container; as we saw, it's easy to attach to a container and then kill it, by issuing a `CTRL+C` to the PID 1 process you've attached to.
