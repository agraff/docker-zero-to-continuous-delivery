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

$ docker rmi <image>
Remove and image.


Use image "busybox" for exercises, it is a very lightweight image.

$ docker run <image> <command>
The image will be pulled automatically if it is not already on the local machine.
	e.g $ docker run busybox sh -c "echo Hello World"

$ docker ps
Shows running containers.

$ docker ps -a
Shows containers that have run previously.


$ docker start <container id, or partial id>
	e.g. $ docker start 2ca

$ docker logs <container id, or partial id>

$ docker rm <container id, or partial id>


## Creating Docker Image

$ docker run -d --name <container name> <image name> <command>
	run as daemon
	e.g. $ docker run -d --name looper ubuntu /bin/sh -c "while true; do echo Hello World; sleep 1; done"

$ docker logs -t <container name>

$ docker stop <container name>

$ docker ps -a


## Binding network port

Do this so can communicate with container, as container does not have ports open to the outside by default.

$ docker run -d -P <image name> <command>
	expose a single random port to the public
	e.g. $ docker run -d -P training/webapp python app.py

$ docker ps
	will show the open port(s)

$ docker run -d -p 80:5000 <image name> <command>
	port 5000 in container is exposed on local (host) machine as port 80


$ docker top <container name>

$ docker ps -a --filter status=exited -q | xargs docker rm
	remove all exited containers


## Working with images

$ docker images
	list all the images that are stored on the host system

$ docker rmi <image_repository:tag>
	e.g. $ docker rmi busybox:latest

$ docker pull <image name>
	download an image, without running it
	offical images are under "library/"
	e.g docker pull library/ubuntu:14.04

$ docker search <query>
	e.g. $ docker search centos

An organisation may run their own docker registry, so they don't need to put their images on the public Docker Hub.
To use a registry, specify the host name and port
	e.g. $ docker pull registry.example.com:5000/ubuntu:14.04


## Interactive container

$ docker run -ti <image name> <command>
	e.g. $ docker run -ti busybox sh


$ docker commit -m "comment" -a "author" <container> <image:tag>
	create an image from existing container


## Creating a Dockerfile
### Filename must be "Dockerfile". File is written in Go.

FROM busybox:latest  # create an image based on this image

RUN echo "Another World" > AnotherWorld.txt  # run this command within the container


Save the file (to "Dockerfile"), then can create an image using:
$ docker build .

$ docker build --tag my_busybox:1.2 .


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


