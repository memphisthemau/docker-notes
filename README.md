## Table of Contents
**[Docker](#docker)**<br>
**[Dockerfile](#dockerfile)**<br>
**[Compose](#compose)**<br>
**[Development and Production instances](#dev-and-prod-instances)**<br>

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
docker rm -v $(docker ps -aq -f status=exited)
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

* Prune all _dangling_ images.

```shell
docker rmi $(docker images -f "dangling=true" -q)
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

* `docker-compose` is a wrapper around the `docker` command. `docker-compose build` will read the `docker-compose.yml` file, look for all `services` containing the `build:` statement and run a `docker build` for each one.
* Only builds the images, does not start the containers:

```shell
docker-compose build
```

* Builds the images _if the images do not exist_ and starts the containers for the `services` in the `docker-compose.yml` file. If you make changes to the `Dockerfile` that the image is based on and want the image to inherit the changes, you can just add the `--build` option instead of manually deleting the image and rerunning.

```shell
docker-compose up
```

* Forces a build of the images even when it is not needed (like using the cache).

```shell
docker-compose up --build
```

* Running the _development server_.

```shell
$ docker-compose up --build
Building hello
Step 1/8 : FROM python:3
 ---> 825141134528
Step 2/8 : ENV PYTHONUNBUFFERED 1
 ---> Using cache
 ---> b90b94a5958d
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> 897498d3ff70
Step 4/8 : COPY app.py /app
 ---> Using cache
 ---> dba72def05e5
Step 5/8 : COPY requirements.txt /app
 ---> Using cache
 ---> 2a830a50f513
Step 6/8 : RUN pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org --proxy http://proxy.bloomberg.com:81 -r requirements.txt
 ---> Using cache
 ---> 26a49354bbbe
Step 7/8 : EXPOSE 80
 ---> Using cache
 ---> 326c8e1fb02c
Step 8/8 : CMD python app.py
 ---> Using cache
 ---> 5352b7d87725
Successfully built 5352b7d87725
Recreating flaskapp_hello_1
Attaching to flaskapp_hello_1
hello_1  |  * Serving Flask app "app" (lazy loading)
hello_1  |  * Environment: production
hello_1  |    WARNING: Do not use the development server in a production environment.
hello_1  |    Use a production WSGI server instead.
hello_1  |  * Debug mode: on
hello_1  |  * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
hello_1  |  * Restarting with stat
hello_1  |  * Debugger is active!
hello_1  |  * Debugger PIN: 197-442-924
hello_1  | 10.144.24.78 - - [04/Aug/2018 15:34:37] "GET /how%20are%20you? HTTP/1.1" 200 -
```

* Starts the containers in _detached_ mode and run them in the background. 

```shell
docker-compose up -d
```

* Check if the containers are running.

```shell
$ docker-compose ps
     Name           Command      State         Ports        
-----------------------------------------------------------
flaskapp_web_1   python app.py   Up      0.0.0.0:80->80/tcp 
```

* Check the console logs of the detached containers.

```shell
docker-compose logs
```

* Stop the running containers.

```shell
$ docker-compose ps        
     Name           Command      State         Ports        
-----------------------------------------------------------
flaskapp_web_1   python app.py   Up      0.0.0.0:80->80/tcp 
$ docker-compose stop
Stopping flaskapp_web_1 ... done
$ docker-compose ps  
     Name           Command      State    Ports 
-----------------------------------------------
flaskapp_web_1   python app.py   Exit 0
```

* Delete the stopped containers.

```shell
$ docker-compose ps  
     Name           Command      State    Ports 
-----------------------------------------------
flaskapp_web_1   python app.py   Exit 0         
$ docker-compose down
Removing flaskapp_web_1 ... done
Removing network flaskapp_default
$ docker-compose ps  
Name   Command   State   Ports 
------------------------------
```

## Development and Production instances

* Running a development environment on high-ports and with these files modified to a certain extend.

```shell
.
+-- apps
|   +-- app.py <- Modify for development.
|   +-- Dockerfile <- Modify for development.
|   +-- requirements.txt
|   +-- templates
|   |   +-- index.html
|   +-- uwsgi.ini <- Not required for development.
+-- docker-compose.yml <- Modify for development.
+-- nginx
    +-- Dockerfile <- Not required for development.
    +-- nginx.conf <- Not required for development.
```

```shell
$ grep 5000 apps/app.py    
    app.run(host="0.0.0.0", port='5000', debug=True)
```

```shell
$ egrep "^CMD" apps/Dockerfile              
CMD ["python", "app.py"]
```

```shell
$ cat docker-compose.yml 
version: '2'
services:
    webui:
        build: ./apps
        ports:
            - "5000:5000"

    #nginx:
    #    build: ./nginx
    #    volumes:
    #        - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    #    ports:
    #        - "81:81"
    #    links:
    #        - webui
```

* Go into the project directory and run `docker-compose ps` to see the running containers. In the dev directory you'll be able to see it's running containers only.

```shell
$ docker-compose ps
       Name              Command      State           Ports          
--------------------------------------------------------------------
flaskappdev_webui_1   python app.py   Up      0.0.0.0:5000->5000/tcp 
```

* `docker ps` will show the global pool of running containers.

```shell
$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                    NAMES
2299a9e0015b        flaskappdev_webui    "python app.py"          4 minutes ago       Up 4 minutes        0.0.0.0:5000->5000/tcp   flaskappdev_webui_1
de447f5a75df        flaskappprod_nginx   "nginx -g 'daemon ..."   16 hours ago        Up 16 hours         0.0.0.0:80->80/tcp       flaskappprod_nginx_1
53f46ed54816        flaskappprod_webui   "uwsgi --ini /app/..."   16 hours ago        Up 16 hours         0.0.0.0:3031->3031/tcp   flaskappprod_webui_1
```

* Docker production directory layout and configuration files.

```shell
.
+-- apps
|   +-- app.py
|   +-- Dockerfile
|   +-- requirements.txt
|   +-- templates
|   |   +-- index.html
|   +-- uwsgi.ini
+-- docker-compose.yml
+-- nginx
    +-- Dockerfile
    +-- nginx.conf
```
