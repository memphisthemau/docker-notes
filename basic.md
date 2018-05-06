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
