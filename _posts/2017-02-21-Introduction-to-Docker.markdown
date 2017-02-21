---
layout: post
title:  "Introduction to Docker"
date:   2017-02-21 13:14:27 -0700
tag: "#Tech #Docker #Container"
id: "8"
---

## What is it?
Docker is a tool that wrap your code in a defined environment. It ensure that your code will run the same way regardless of the machine. No more "but it worked on my machine..." 😉.

## Why?
With Docker, you'll make your app easily deployable and portable. In a team, it allows your teammates and you to work on the exact and same environment. Super useful.

For instance, you wanna write a Ruby app. With Docker, you can define an environment from the Ruby base image with ruby already installed. Then you write your code, copy it onto this environment and execute it. Even if Ruby isn't installed on your system, it's embedded on your defined environment and it will run on your machine #magic.

Now think of a large and complex system that you want to deploy on a new server. Deploying using Docker makes it very easy!

## How?

First you need to install Docker on your system (https://docs.docker.com/engine/installation/linux/) for Linux system, here for macOS (https://www.docker.com/products/docker#/mac).

Now lets see how we can use Docker. Docker use containers to run. A container is an instance of Docker that execute processes on a defined environment. To define an environment, you need to start from a base image. Docker provide you with some base image such as an nginx server, an Ubuntu system, a Ruby environment and so on.

### Running your first container

```
docker run -it ubuntu bash
```

There you are, you are now on a Ubuntu system like, on your machine. Some of the basic commands are missing but you can install them using ```apt-get```.

```docker run``` is used to create and start a new container. The ```-it``` option allows you to keep focus on the container and use it as a shell. Then, ```ubuntu``` is the name of the base image that we are going to use. ```bash``` is the command we run on this container. Here we are running a shell instance.

### Port binding

You can redirect the traffic from a given host port to your container port using the ```-p``` option. For instance, lets say that my app is running on port 9000 of my container. I want to bind the port 9000 of my host machine to the port 80 of my container. I just use ```-p HOST_PORT:CONTAINER_PORT```

```
# On your host machine
docker run -p 9000:80 nginx
```

And then open your favorite web browser and go to http://localhost:9000/. You should see the nginx default page.

### Demonize the container

We ran a nginx container in foreground in the previous example. Most of the time, we like that our web server run as a daemon. To do so, you can start containers as daemon using the ```-d``` option. Now, you can run

```
# On your host machine
docker run -d -p 9000:80 nginx
```

Docker answer with a long id. It's the unique id of your container. Now lets go into the system. We are going to execute a bash process onto our nginx container. Then we will create a simple web page directly onto the container. But how do we access our container while it's already running?

```
# On your host machine
docker exec -it <CONTAINER_ID> bash
```

We now have access to a shell running on our nginx container. Lets go in the ```/usr/share/nginx/html``` repertory. Install your favorite shell text editor and create a small html page in test.html.

Then go to http://localhost:9000/test.html you should see your awesome web page.

### Sharing files between Host and Container

Most of the time, to deploy your website, you won't start an nginx container and then write your pages on it as we just did! You can either copy files onto the container (explained below) or you can share files between the host machine and your container.

To do so, just use the ```-v``` option as follow : ```-v host_dir:container_dir```.

For instance, go to your home dir on your host machine and run the following :

```
# On your host machine
docker run -it -v $PWD:/host ubuntu bash
```

And then run ```ls```. You should see a ```host``` repertory. Now run

```
# On your container
ls /host
```
 you should see every files that are in your home directory of your host machine. You are now sharing files between your host machine and your ubuntu container. Be careful, you can add modify or remove file on your container, it will also change your host directory, these are the same files, this is not a copy!!!

Lets say you want to run a container for your website, just use the following :

```
# On your host machine
cd /to/your/project/folder
docker run -d -p 9000:80 -v $PWD:/usr/share/nginx/html/ nginx
```

And you can access your website at http://localhost:9000/.

### Manage your containers

You can see all your running containers on your host machine using :

```
docker ps
```

Kill or stop your containers using:

```
docker stop <CONTAINER_ID>
# Or
docker kill <CONTAINER_ID>
```
When you kill or stop a container, it will go in the "stop" state. You can see all containers (running and stopped) using the ```-a``` option. You can bring back a stopped container using:

```
docker start <CONTAINER_ID>
```

Or you can delete a container using:

```
docker rm <CONTAINER_ID>
```

### Dockerfile

You can write a script in order to define your container environment. This script is called ```Dockerfile```. For instance, you saw that on the Ubuntu official image, some basic commands are missing and you have to install it using ```apt-get install```.

Lets create a file, call it "Dockerfile". And write the following into it:

```
FROM ubuntu
MAINTAINER your_name VERSION 1.0

# Execute a command on the new container
RUN apt-get update

# When using apt-get install don't forget the -y option otherwise it times out
RUN apt-get install netcat -y
RUN apt-get install nano -y
RUN apt-get install vim -y
RUN apt-get install iputils-ping -y

#The command we are going to execute when running our container
CMD bash
```

And then, we are going to build an image named "my_ubuntu" from your Dockerfile:

```
docker build -t my_ubuntu .
```

Now, check that you have an image called "my_ubuntu":

```
docker images | grep ubuntu
```

You should see at least two lines. One image called "ubuntu" which is the official image that we use previously and your new image "my_ubuntu". Now create and run a container using my_ubuntu:

```
docker run -it my_ubuntu
```

You don't need to add a "bash" since we mentioned it in our Dockerfile. You are now running a shell instance on a Ubuntu with ```ping```, ```nano```, ```vim``` installed 😊.

### Create an image for your website

Create a ```Dockerfile``` in your website path.
```
FROM nginx
MAINTAINER your_name VERSION 1.0

# Copy your website resources from your host
COPY ./ /usr/share/nginx/html/

# Open the container port on which your app is running
EXPOSE 80
```

And run

```
docker build -t my_website .
```

And finally

```
docker run -d -p 8080:80 my_website
```

And go to http://localhost:8080/ to see your website.


## Where to go from here

You know how to manipulate containers and how to create custom images using Dockerfile. A lot of images have already been created. You can find them there (https://hub.docker.com). You can deploy simple application with that, but how do we deploy an app that use a database and a REST API and make those two instance communicate? We have to use ```docker-compose``` to do so. I will write an article on this topic A.S.A.P.
