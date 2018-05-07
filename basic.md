# Pull and run an image from a local repository

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


