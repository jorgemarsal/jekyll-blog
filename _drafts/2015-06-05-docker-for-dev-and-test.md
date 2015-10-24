---
layout: post
title: "Docker for dev and test"
description: ""
category:
tags: []
---

CTO/Engineering manager -> how to improve the efficiency of your organization by using docker

Intro to value of Docker.
Mention Kubernetes

In this example we are going to create a Centos 7 image running an SSH service. First we create a `Dockerfile`:

```
FROM centos:centos7

# install required packages
RUN yum -y install vim openssh-server sudo glibc tar openssh-clients initscripts

# create user
RUN useradd  --create-home jorgem
RUN mkdir -p /home/jorgem/.ssh/
ADD id_rsa.pub /home/jorgem/.ssh/id_rsa.pub
ADD id_rsa /home/jorgem/.ssh/id_rsa
RUN cat /home/jorgem/.ssh/id_rsa.pub >> /home/jorgem/.ssh/authorized_keys2
RUN chown -R jorgem /home/jorgem
RUN echo "jorgem ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# configure SSH
ADD sshd_config /etc/ssh/sshd_config
RUN mkdir /var/run/sshd
RUN echo 'root:screencast' | chpasswd
RUN /sbin/sshd-keygen

# expose port and start sshd daemon
EXPOSE 22
CMD    ["/usr/sbin/sshd", "-D"]
```

Next we create a Docker image:
```
cp ~/.ssh/id_rsa* .
sudo docker build -t centos7 .
```

Now we can spin a small cluster of 4 Docker containers:

```
run -d --name centos7-000 centos7
run -d --name centos7-001 centos7
run -d --name centos7-002 centos7
run -d --name centos7-003 centos7
```

To connect to the containers we need to get their IP addresses:
```
docker_ip() {
    sudo docker inspect $1 | grep IPAddress | cut -d '"' -f 4
}
```

```
$ docker_ip centos7-000
172.17.0.18
$ ssh 172.17.0.18
```
