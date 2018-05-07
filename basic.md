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

