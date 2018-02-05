---
title: "Using both Docker and Virtualbox on Windows"
path: "/using-both-docker-and-virtualbox-on-windows/"
date: "2018-01-30T22:13:00.000Z"
date_updated: "2018-02-01T22:13:00.000Z"
tags: windows, docker, virtualbox
---

Oracle VirtualBox is a free and open-source hypervisor for x86-64 computers currently being developed by the Oracle Corporation.
Microsoft Hyper-V, codenamed Viridian and formerly known as Windows Server Virtualization, is a native hypervisor; it can create virtual machines on x86-64 systems running Windows.
The two are not exactly the same kind of hypervisor but we won't get into that now.

It turns out that when Hyper-V is installed on your Windows box, the hypervisor is running all the time underneath the host OS.
As only one thing can control the VT hardware at a time (for stability), the hypervisor blocks all other calls to the VT hardware.
In other words, Hyper-V must be turned off in order to use VirtualBox.

The nasty part is that Docker actually needs Hyper-V in order to run on Windows. 
To fix that, there are two options, so you can get the best out of both worlds:


# Toggling Hyper-V

You can remove Hyper-V from you system via `Turn Windows features on or off` and reboot your system.
As far as I know, this actually (un)installs Hyper-V from your computer and doing so repeatedly might cause you to end up with a registry mess.

You can disable Hyper-V without uninstalling it by starting a command prompt with administrator rights and executing the following command:
```
bcdedit /set hypervisorlaunchtype off
```

If you want to enable it, then run this command:
```
bcdedit /set hypervisorlaunchtype auto
```

This solution still needs a reboot, but you don't mess up your registry and don't lose configuration settings.


# Making use of Docker-Machine

In the early days (pre Docker v1.12) Docker didn't run on native Mac or Windows OS, so another tool was created, Docker-Machine.
Docker-Machine creates a virtual machine (using yet another tool, e.g. Oracle VirtualBox), runs Docker on that VM, and helps coordinate between the host OS and the Docker VM.

You can check if you have Docker-Machine installed by running the following command:
```
$ docker-machine -v
docker-machine.exe version 0.13.0, build 9ba6da9
```

If the output is not similar to the one above, install it by running the following command as described [here](https://docs.docker.com/machine/install-machine/#install-machine-directly):

```
$ if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"
```

Now that we have installed docker-machine we are able to create a new environment.
In case you already had docker-machine installed we can first have a look on what machines we have available using the following command:
```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default   *        virtualbox   Running   tcp://192.168.99.100:2376           v18.01.0-ce
```

In case you already have one or more machines present you might want to use one of those.
To create a new Docker machine run the following command:
```
$ docker-machine create --driver virtualbox default # where 'default' is the new machine name
Running pre-create checks...
(default) Image cache directory does not exist, creating it at C:\Users\vrachieru\.docker\machine\cache...
(default) No default Boot2Docker ISO found locally, downloading the latest release...
(default) Latest release for github.com/boot2docker/boot2docker is v18.01.0-ce
(default) Downloading C:\Users\vrachieru\.docker\machine\cache\boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.01.0-ce/boot2docker.iso...
(default) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(default) Copying C:\Users\vrachieru\.docker\machine\cache\boot2docker.iso to C:\Users\vrachieru\.docker\machine\machines\default\boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to create a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(default) Found a new host-only adapter: "VirtualBox Host-Only Ethernet Adapter #2"
(default) Windows might ask for the permission to configure a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe env default
```

Since Docker isn't running on your actual host OS, docker-machine needs to deal with IP addresses and ports and volumes and such. 
Its settings are saved in environment variables, which means you have to run the following command every time you open a new shell:
```
eval $(docker-machine env default)
```

Thus far I've assumed that you're using Bash/Cygwin or some other shell emulation tool.
If you want to use CMD or PowerShell just run the following and follow the instructions profided on the last lines outputted:
```
$ docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="C:\Users\vrachieru\.docker\machine\machines\default"
export DOCKER_MACHINE_NAME="default"
export COMPOSE_CONVERT_WINDOWS_PATHS="true"
# Run this command to configure your shell:
# eval $("C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env default)
```

Now we'll be able to use Docker on Windows again.
Lets try using the `docker/whalesay` image to test our Docker environment.
```
$ docker run docker/whalesay cowsay "Awesome! Your Docker environment is running oldschool, without Hyper-V."
/ Awesome! Your Docker environment is \
\ running oldschool, without Hyper-V. /
 -------------------------------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```