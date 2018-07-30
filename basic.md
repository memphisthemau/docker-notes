## Table of Contents
**[Docker](#docker)**<br>
**[Dockerfile](#dockerfile)**<br>
**[Compose](#compose)**<br>

## Docker
* Runs an image from the local repository and exits immediately.

```shell
munheng@ubuntu:~$ sudo docker run localhost:5000/local-centos
munheng@ubuntu:~$ 
```

* Runs an image from the local repostory and gets an interactive TTY shell.

```shell
munheng@ubuntu:~$ sudo docker run -i -t localhost:5000/local-centos 
[root@6bbf8236255a /]# 
```

* Run an image from the local repository. Start with __-d__ and __-it__ to run in daemon mode and be able to reattach.

```shell
munheng@ubuntu:~$ s docker run -d -it localhost:5000/local-centos bash
000c8a767f2754ac1ad6cbf62c85922b79cf85a2ea163eda888d45a78d568581

munheng@ubuntu:~$ s docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                    NAMES
000c8a767f27        localhost:5000/local-centos   "bash"                   4 seconds ago       Up 3 seconds                                 dazzling_hodgkin
d51ca36e5f42        registry:2                    "/entrypoint.sh /etc…"   37 hours ago        Up 37 hours         0.0.0.0:5000->5000/tcp   local-repository
```

* Reattach to the instance running in the background. _Exits_ and stops the running instance.

```shell
munheng@ubuntu:~$ s docker attach 00
[root@000c8a767f27 /]# exit
```

* Reattach to the instance running in the background. __Ctrl-p__ + __Ctrl-q__ detaches from instance and leaves it running in the background.

```shell
munheng@ubuntu:~$ s docker attach 00
[root@000c8a767f27 /]# read escape sequence 
```

* Find the IP address of a running container.

```shell
munheng@ubuntu:~$ s docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d72a1dd7e14e        debian              "sleep 3600"        5 minutes ago       Up 5 minutes                            trusting_hoover

munheng@ubuntu:~$ s docker inspect d7 | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
                    
munheng@ubuntu:~$ ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.081 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.100 ms
^C
--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1009ms
rtt min/avg/max/mdev = 0.081/0.090/0.100/0.013 ms

munheng@ubuntu:~$ s docker stop d7
d7

munheng@ubuntu:~$ s docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

munheng@ubuntu:~$ s docker inspect d7 | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "",
```

### Cleaning Up

* Cleaning up stopped containers.

```shell
$ docker rm -v $(docker ps -aq -f status=exited)
```

* Kill all running containers.
```shell
docker kill $(docker ps -q)
```

* Delete all stopped containers.

```shell
docker rm $(docker ps -a -q)
```

* Delete all images.
```shell
docker rmi $(docker images -q)
```

## Dockerfile

* **FROM** instruction specifies the base image to use. All `Dockerfiles` must have a **FROM** instruction as the first non-comment instruction. 
* **RUN** instructions specify a shell command to execute inside the image.
```shell
$ cat Dockerfile 
FROM ubuntu

RUN apt-get update && apt-get install -y python python-pip

RUN pip -v install --trusted-host=pypi.org --trusted-host=files.pythonhosted.org --proxy=http://proxy.domain.com:81 flask

COPY app.py /opt/

ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0 --port=80
```
* *Build* the image by running `docker build` inside the same directory where the `Dockerfile` lives. Use `--no-cache` to do a clean build without relying on the cache from the last build.
```shell
docker build --no-cache --build-arg http_proxy=http://proxy.domain.com:81 -t my-simple-webapp .
```
* *Run* the image by typing `docker run -it <image_name:tag> [command_to_be_executed_on_image]`.

```shell
$ docker run -it mydjango:v0.1
root@75d08980aade:/# 
root@75d08980aade:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@75d08980aade:/# exit
```
* Start the image as a *daemon* and connect to the running container.

```shell
$ docker run -it -d mydjango:v0.1
574da457a562e129027c07fb0acfc21eab55e9d0529f0b2a856525586bd45e37
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS           PORTS     NAMES
574da457a562        mydjango:v0.1       "/bin/bash"         13 seconds ago      Up 13 seconds              hopeful_hypatia
$ docker attach 574da457a562
root@574da457a562:/# 
```

## Compose
