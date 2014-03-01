# Introducing 'docker-autocommit'

'docker-autocommit' is a PoC (Proof Of Concept) developed in pure 'bash' that sets my workflow.

It just spawns an interactive 'bash' shell on a given 'docker' "image:tag" and starts to monitor its '.bash_history' which is configured to inmediately write commands to disk. They are written once they have exited, wether exited _zero_ or not!

It monitors the '.bash_history' file in the container with 'inotifywait' and when a change is detected, a "RUN _command_executed_in_the_interctive_shell_" is added to a _Dockerfile.$timestamp_ in the directory where we run 'docker-autocommit'.

Now with 'docker-autocommit' you have an easy way to write your _Dockerfiles_ without too much _copy & pasting_.

DISCLAIMER: 'docker-autocommit' needs 'root' permissions in order to access to containers' '.bash_history' files. But it's just through 'inotifywait' and 'tail' executions, no other code is "sudoed" :]

## Why?

I haven't seen too many proposals on how to develop _Dockerfiles_, so I just went for my own.

I think it's easier to model a container interactively, automagically writing a complete _Dockerfile_ and stripping the not needed parts than writing one from nothing and then picking up just the commands you decide to add. It just didn't seemed agile for me.

Think about it like the inverse approach to write a _Dockerfile_.

### Why 'bash'?

Because from the PoV of having the minimum dependencies (and maximum cross-distro support), 'bash' and the 'inotify-tools' suite looked good.

But I have some other branch around to implement it in ruby.

### Why not contribute to 'docker' project directly?

Because this is just a PoC :]

# Installation

Install depends:
```
apt-get install -y git curl inotify-tools || yum install git curl inotify-tools
```

Clone repository and link executable:

```
git clone https://github.com/rcmorano/docker-autocommit.git
sudo ln -s $PWD/docker-autocommit/bin/docker-autocommit /usr/local/sbin/
chmod +x /usr/local/sbin/docker-autocommit
```

Or just download the binary:

```
sudo curl -L https://raw.github.com/rcmorano/docker-autocommit/master/bin/docker-autocommit -o /usr/local/sbin/docker-autocommit
sudo chmod +x /usr/local/sbin/docker-autocommit
```

## Okay, what's the flow?

Create your container development dir:
```
rcmorano@localhost:~$ mkdir -p Projects/container-poc
rcmorano@localhost:~$ cd Projects/container-poc
rcmorano@localhost:~/Projects/container-poc$ 
```

Spawn an interactive shell with 'docker-autocommit':
```
rcmorano@localhost:~/Projects/container-poc$ docker-autocommit ubuntu:saucy
We need 'root' privileges for some actions!
[sudo] password for rcmorano: 
root@1968e940db10:/# 
```

Install some packages:
```
root@1968e940db10:/# apt-get update
_OUTPUT_REMOVED_
root@1968e940db10:/# apt-get install -yo 'APT::Install-Recommends=false' -o 'APT::Install-Suggests=false' curl vim
_OUTPUT_REMOVED_
root@1968e940db10:/#
```

Finish container execution by exiting:
```
root@1968e940db10:/# exit 0

exit
rcmorano@localhost:~/Projects/container-poc$
```

Get a ready to build _Dockerfile_:
```
rcmorano@localhost:~/Projects/container-poc$ ls
Dockerfile.201402222035
rcmorano@localhost:~/Projects/container-poc$ cat Dockerfile.201402222035 
FROM ubuntu:saucy
RUN apt-get update
RUN apt-get install -yo 'APT::Install-Recommends=false' -o 'APT::Install-Suggests=false' curl vim
RUN exit 0

```
_NOTE: if you exit from shell with CTRL+D the 'exit 0' won't be saved to Dockerfile_

## Considerations

* You could dettach and commit your interactive container since 'docker-autocommit' will clean the only two files it produces but...:
* IMPORTANT: Note that if you make any changes to filesystem from outside command line (in e.g.: you edit files with 'vim' or modify anything interactively), your _Dockerfile_ might build an inconsistent container not reproducing the exactly same container as the one of the interactive shell.
* You can edit the generated _Dockerfile_ while running the interactive shell, but be careful not to overwrite changes made by 'docker-autocommit'.
* More to add, suggestions accepted

# Contrib!

_#FIXME#_

Just run a 'docker-autocommit' developement container (it's a trusted build linked to this project's _Dockerfile_):

```
git clone https://github.com/rcmorano/docker-autocommit.git
# -v is used to mount git repo inside the dev environment and
# -privileged is used because 'docker' is run inside the dev container for bdd tests
docker run -v $PWD/docker-autocommit:/docker-autocommit -privileged -t -i rcmorano/docker-autocommit-dev-env
```
And you'll get a pristine developer environment with a separated 'git' repo mounted inside the container to start implementing its features:

```
WARNING: WARNING: Local (127.0.0.1) DNS resolver found in resolv.conf and containers can't use it. Using default external servers : [8.8.8.8 8.8.4.4]
root@c29ae4ab011d:/# cd docker-autocommit
root@c29ae4ab011d:/docker-autocommit# cucumber
_#TODO#_
...
```

*Note* that the dev-container is configured to run docker with the experimental 'btrfs' driver. Just remove '-s btrfs' from 'Dockerfile' and rebuild container if you are using pure 'aufs'.
*Note2* 'aufs' use on top of 'btrfs' is broken AFAIK 

## Why this way of development?

* You can setup development environment in seconds
* They will be pristine ones
* So no temporals or extra depends. It will force you to update the development container in case you installed something last time and forgot to commit changes pushing you in the right way.
* You can get a new developer to pass some features for the project with a simple 'docker run' 
* Any team member could 'docker build' and upload the container to registry safely
* It's easy, clean and fast

## Considerations

* If you use 'aufs' and you'd like to have the outside-docker daemon images available to the dev-container so you do not have to redownload 'ubuntu' every time we run a development container, you should mount '/var/lib/docker' inside the container
  * Note also that you may not have a pristine environment for development and testing this way

# License and Author

Author:: Roberto C. Morano (<rcmova@gmail.com>)

Copyright:: 2014, Roberto C. Morano (<rcmova@gmail.com>)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
