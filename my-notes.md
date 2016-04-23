Docker: Zero to Continuous Delivery
===================================

What happens when you set up a container:
Pull image
Prepare container
  Prepare filesystem
  Prepare network
  Run application

Docker Commands
---------------

$ docker version
Versions are not backwards compatible. Client and Server must have the same Version and API version.

$ docker info
Information about the Docker daemon.

$ docker help <command>

$ docker images
List images on the local machine.

$ docker pull <image>
<image> has the format: [user]/repository[:tag]
[user] is optional, defaults to library.
tag is optional, defaults to latest.
For all offical images, user=library, e.g. library/ubuntu
Gets images from hub.docker.com (Docker Hub)
A single image can have multiple tags assigned to it - see Docker Hub to find the tags.

$ docker search <query>
e.g. $ docker search centos

$ docker rmi <image>
Remove an image.

An organisation may run their own docker registry, so they don't need to put their images on the public Docker Hub.
To use a registry, specify the host name and port
	e.g. $ docker pull registry.example.com:5000/ubuntu:14.04




Use image "busybox" for exercises, it is a very lightweight image.

$ docker run [--name <name>] [--label <label>] <image> [command]
The image will be pulled automatically if it is not already on the local machine.
	e.g $ docker run busybox sh -c "echo Hello World"
<name> must be unique

$ docker ps
Shows running containers.

$ docker ps -a
Shows containers that have run previously.

$ docker inspect <container or image>
See information inside the container.
<container> is the full container id or partial container id.

$ docker start <container>
Start a stopped container.

$ docker logs <container>
Show the logs of a container - anything written to stdout and stderr for that container.
You can use `docker inspect` and look for the `LogPath` property - this shows the location of the logs in the file system of the server running the docker daemon. It is a good idea to have the logs saved to a different volume to the root volume, or you may fill up your root volume and break the entire system.
-t : show timestamps
-f : follow tail
--tail <n> : show last n lines

$ docker rm <container>
Removes everything for that container, including its logs.


## Run container in background

$ docker run -d --name looper ubuntu /bin/sh -c "while true; do echo Hello World; sleep 1; done"
-d : run as a daemon (in the background)

$ docker rm looper
!! This will fail because the process is still running in the container. Need to use:
$ docker rm -f looper
This sends SIGKILL.

Better to use:
$ docker stop looper
This sends SIGTERM (then SIGKILL after a time period).

$ docker kill <container>
Kill a running container
  -s, --signal=KILL




## Running a web application inside docker

$ docker run -d training/webapp python app.py


### Binding network port

Do this so can communicate with container, as container does not have ports open to the outside by default. Need to stop the container before being able to change port binding.

$ docker run -d -P <image> <command>
	expose a random port to every exposed port in the image (see further down for how to expose ports from an image)
	e.g. $ docker run -d -P training/webapp python app.py

$ docker ps
Will show the open port(s), e.g. 0.0.0.0:32768->5000/tcp , then can call the web app:
$ curl localhost:32768

$ docker run -d -p 80:5000 <image> <command>
Port 5000 in container is exposed on local (host) machine as port 80.
You can run multiple containers from the same image, mapping each container to a different local port.




$ docker top <container>
Display the running processes of a container.

$ docker ps -a --filter status=exited -q | xargs docker rm
Remove all exited containers.

$ docker ps -a -q | xargs docker rm -f
Remove all containers.

$ docker run -ti <image> <command>
Run a container in interactive mode. e.g. $ docker run -ti busybox sh
Can create a file inside this container, then create an image from it (see below) then create a container from that new image and the new container will have the file inside it.

$ docker commit -m "comment" -a "author" <container> <image:tag>
Create an image from existing container


## Creating a Dockerfile
Dockerfiles are written in Go. The default filename is "Dockerfile".
Instructions (directives) are written in ALL CAPS.
Create a directory (e.g. my_busybox) then change to that directory. Create a file named `Dockerfile` with the following contents:

FROM busybox:latest  # create an image based on this image
RUN echo "Hello World" > hello-world-2.txt  # run this command within the container

Save the file, then can create an image using:
$ docker build .
This will create an image, using Dockerfile, but not give it a name or tag. You can name and tag with:
$ docker tag <image> <name>:<tag>
e.g. $ docker tag ab3d2 my_busybox:v2

Or name and tag when creating the image:
$ docker build --tag my_busybox:1.2 .


### COPY Instruction
COPY <source> <destination>
e.g.
COPY AnotherFile.txt /AnotherUnecessaryFile.txt

Can also use ADD in some cases. See documentation for the difference.

### CMD Instruction
CMD <command_array>
command_array must be of the form: ["arg0", "arg1", "arg2", ...]
e.g.
CMD ["sh", "-c", "cat /AnotherUnecessaryFile.txt"]


### WORKDIR Instruction
Set working direction (current directory from that point onwards)







### ENTRYPOINT vs COMMAND

#### Entrypoint

FROM busybox:latest
RUN echo "Another World" > AnotherWorld.txt 
ENTRYPOINT ["sh", "-c", "echo Hello World"]  # need to separate each argument by comma

If you add a command (e.g. from 'docker run' command), it is appended to the command specified with ENTRYPOINT


FROM busybox:latest
RUN echo "Another World" > AnotherWorld.txt
CMD ["-c", "echo Not Done"]
ENTRYPOINT ["sh"]

The command (CMD) is appended to the entry point. Both are specifying the meta data for the image (so order of declaring CMD and ENTRYPOINT does not matter).

Both CMD and ENTRYPOINT can be overwritten with the docker command.
$ docker run -ti <image> <command>


## Storing Images in Private Registry

1. Name the image properly
	$ docker tag <image> <name for image in repository>
		e.g. $ docker tag my_busybox registry.training.local/busybox:1
2. Push to the repository
	$ docker push registry.training.local/busybox:1

## Use the registry API

(You can use a Docker image to create a registry)

$ curl -sL registry.training.local/v2/ | jq .
	check the registry is running (returns {})

$ curl -sL registry.training.local/v2/_catalog | jq .
	get list of images

*** How to get JSON that described the container (showed ports, etc)







Docker is always running as root! This is so it has privilege to do the things it needs to do. Latest version does improve this a bit.


