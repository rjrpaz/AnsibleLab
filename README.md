# AnsibleLab
Ansible controller and node, Fedora based, to deply and test Ansible Playbooks

For Windows 10 users interested in using docker, please check for *Docker Toolbox* (https://docs.docker.com/toolbox/toolbox_install_windows/).

This project includes necessary files to develop docker images for Ansible controller and nodes.

This images are Fedora based, and can be builded as follow:

**Create ssh keys, that way controller can access nodes without user validation**
ssh-keygen -t rsa -f files/id_rsa

**Ansible Controller**:
(without proxy support)

docker build -f Dockerfile.Controller -t ansiblecontroller .

(with proxy support)

docker build --build-arg proxy=<http://proxy_ip:proxy_port> -f Dockerfile.Controller -t ansiblecontroller .

**Ansible Node**:
(without proxy support)

docker build -f Dockerfile.Node -t ansiblenode .

(with proxy support)

docker build --build-arg proxy=<http://proxy_ip:proxy_port> -f Dockerfile.Node -t ansiblenode .

Pre-builded images can be downloaded from Docker hub:

docker pull rjrpaz/ansiblecontroller

docker pull rjrpaz/ansiblenode



## TESTING AND DEPLOYING PLAYBOOKS

Once docker images are available, must be runned in the following sequence:

1) Create a shared network for ansible containers.

docker network create ansiblenetwork

2) Running ansible node container in background. It can be accesed by SSH.

docker run -d --name node1 --network=ansiblenetwork ansiblenode

3) Running ansible controller in foreground. In this case, we are going to share a directory with host machine, so we can save file modifications in a non-volatile environment.

(Windows. We asume that GitHub project was cloned on upper directory level from Docker Toolbox)
docker run -it --network=ansiblenetwork -v /c/Users/`whoami`/Project_Anycast/ansible:/ansible ansiblecontroller /bin/bash

(or Linux)
docker run -it --network=ansiblenetwork -v `pwd`/ansible:/ansible ansiblecontroller /bin/bash

Now we can run ansible commands. We can test ansible reachability:

ansible -i /ansible/hosts -m ping -u admuser node1


## FILES

* Dockerfile.Controller: Dockerfile for bulding Ansible controller image.
* Dockerfile.Node: Dockerfile for bulding Ansible controller node.
* Readme.md: This file
* ansible/: directory that includes ansible files and playbooks.


## BUGS
When installing iputils package using dns, I get the error message:

  Installing       : iputils-20161105-7.fc27.x86_64                       13/13 
error: unpacking of archive failed on file /usr/bin/ping;5a8b618f: cpio: cap_set_file failed - Inappropriate ioctl for device

this seems to be was caused by AUFS. This could be fixed adding this option in file /etc/default/docker

DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4 -s devicemapper"

and also modifying /lib/systemd/system/docker.service

[Service]
EnvironmentFile=/etc/default/docker
ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS

