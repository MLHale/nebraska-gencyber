# Containers - A primer

## Think shipping containers
- Standard unit of shipping.
- Stackable onto on a ship, train, plane or truck. ⛴️ 🚇 ✈️ 🚚
- Compatible worldwide.

## Enter application containers
- Standard unit of an application’s runtime.
  (files, libraries, dependencies, sockets, etc)
- Stackable onto a Linux, macOS or Windows Machine.
- Can run anything that would run on a server.


----------
# Cybersecurity and Containers

## Domain Separation

- Separate application environment
- Separate source code from runtime

## Modularization
- Separate services from each other
  (Web service, Database, Application Server, etc)

## Resource Encapsulation
- Standard interface between the
  container and host or other containers
- Expose legitimate application interfaces only

## Process Isolation
- Limit damage/crashes to a single container
- Processes in a container cannot access
  resources in another container directly


----------
# Introduction

## Lesson goals
- Deploy, run and publish a container
- Manage container interactions
- Use containers to setup a development environment
- Network containers

## Materials required
- [Docker CE (Community Edition)](https://docs.docker.com/engine/installation/#supported-platforms)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Prerequisite lessons
- Github Tutorial
- [Linux Commands](https://www.cheatography.com/davechild/cheat-sheets/linux-command-line/pdf/)

---

# Table of contents

[Deploy, run and publish a container](#deploy,-run-and-publish-a-container)  
[Manage container interactions](#manage-container-interactions)   
[Setting up a dev environment](#setting-up-a-dev-environment)   
[Acknowledgements](#acknowledgements)   
[License](#license)  


----------
# Deploy, run and publish a container
## Your first container

Let’s start with a container based on [Alpine Linux](https://alpinelinux.org)

- Alpine Linux is a security-oriented, lightweight Linux distribution

First we need to download a container blueprint called an Image

- Open the Command Line Interface (CLI) i.e.   
  Terminal (Linux, macOS) or Command Prompt (Windows)

```bash
# Download alpine container image from Docker Hub
docker pull alpine
```

## Container images

[Docker hub](https://hub.docker.com) is a registry of images from authoritative sources and individual users

- By default the `latest` image is downloaded. This label is called a Tag
- Other tags allows downloading specific versions or those shared by other users

Let’s check locally available images  

```bash
# Check images available locally on your machine
docker images
```

> Observation: Docker images are much smaller than typical Virtual Machines  

## Running your first container

Let's create and start a container from the alpine image
```bash
# See list of docker commands
docker
# `run` command creates and start a new container
# -it provides an interactive tty shell into the container
# --name provides a name for your new container
# `alpine:latest` is the image name and its tag
docker run -it --name myAlpine alpine:latest
```

## Explore the container

Try some commands in the container shell
```bash
# Some commands to try
whoami # enuf said!
cd /   # Switch to the root directory
ls -la # List contents of the root directory
ping google.com # Hit CRTL+C to exit
exit # Stop the shell to exit container
```

## Observations
- Host directories and files are inaccessible
- The container can connect to websites and receive responses
- You were running as `root` in the container!

# This is so cool… 😎

[Back to TOC](#table-of-contents)

----------
# Manage container interactions

## Container volumes
- Containers are short-lived
- Code and data persist over longer periods
- Volumes are externally mapped storage areas for a container
  - Shared folders on host, Shared drives on network, etc.,

## Mount a host directory as a data volume

Volumes have to be initialized at container creation time
- The `-v` option mounts a volume

In a **host** shell:
```bash
# Make sure the host directory exits, otherwise create it
mkdir /src/webapp
# Host /src/webapp folder mapped to container /webapp folder
# `\` allows you to continue a long command on a new line
docker run -it --name myAlpineWithVol \
-v /src/webapp:/webapp alpine:latest
```

## Volume configuration

Caution 😡:   
By default a mounted volume allows full read/write by the container  
This allows exceptions to the `Process Isolation`  principle

- May set it to read-only (Least privilege principle)
  - `-v /src/webapp:/webapp:ro`
- Caching option improves performance
  - `-v /src/webapp:/webapp:cached`

## Test the mounted volume

In the **container** shell:
```bash
# Change directory to the `/webapp` directory in the container
cd /webapp
# Create a new file with a simple message
echo "#NebraskaGencyberRocks" > test.txt
```

In a **host** shell (open another host terminal):
```bash
# Change directory to the `/src/webapp` directory on the host
cd /src/webapp
# List contents of the test.txt file
cat test.txt
# Overwrite the file and add exclamation marks
echo "#NebraskaGencyberRocks!!!" > test.txt
```

Back in the **container** shell:
```bash
# List contents of the test.txt file
cat test.txt
# You should be able to see the new exclamation marks
```

> Observation: Container and host are able to share files.

# Totally cool… 🤓

## Exposing Container Services

Services are bound to container ports.
We need to expose container ports to the network to access these services remotely.

## A HTTP Server Container
Let’s create a container that runs an HTTP server in two commands!
First, download an image for Lighttpd from Docker Hub

```bash
# download a container for lighttpd, a lightweight HTTP server
docker pull gists/lighttpd
```
In the spawned container, we need to expose Port 80 to access the web server.  
We do this by mapping the container’s port to a port of the host

## Host - Container Network
![Host-container network](../img/containers/host-container-network.png)

## Host-Container Port Mapping

Port mappings have to be initialized at container creation time
- The `-p` option maps a host port to the container port

In a `host` shell (open another host terminal)

```bash
# -d option runs the container in daemon mode (background)
# -p 8888:80 maps host port 8888 to container port 80
docker run -d --name lighttpd \
           -p 8888:80 \
           -v /src/webapp:/var/www \
           gists/lighttpd
```

## Test the mapped port

In the previous `host` shell:

```bash
# Change directory to the `/src/webapp` directory on the host
cd /src/webapp
# Add a simple HTML file exclamation marks
echo "<html>My first Container App</html>" > index.html
```
Browse to http://localhost:8888

## Testing continued…

Return to the `host` shell
```bash
# Update the HTML file
echo "<html><h1>Cool</h1></html>" > index.html
```
Revisit http://localhost:8888

## Reflect on what just happened 🤔

> Observations:
- Separation of persistent code from the application runtime
- Host file updates are instantly reflected in the container application

# 😎 Cool!
[Back to TOC](#table-of-contents)

----------

# Setting up a dev environment

## Container build automation
Typing long docker commands in a terminal is cumbersome 😖  
Luckily, a `Dockerfile` automates the build process. This is Akin to a “recipe” that the Docker engine understands.

## Dockerfile
- Examine the `Dockerfile` for `gists/lighttpd` image that we pulled from Docker Hub earlier: https://github.com/iHavee/dockerfiles/blob/master/lighttpd/Dockerfile

- Here is a reference for Dockerfile directives: https://docs.docker.com/engine/reference/builder/

## A Django Dockerfile

```bash
# Start with this base image
FROM python:2.7.13
# Set evironment variables
ENV PYTHONUNBUFFERED 1
# Set the working directory in
# which RUN and CMD options will run
WORKDIR /var/www/backend
# RUN commands run at container build time
# Used to install applications
RUN pip install Django
RUN pip install djangorestframework
RUN pip install markdown
RUN pip install django-filter
RUN pip install psycopg2
```

## Basic automation workflow

### Step 1: Clone a repository
Let us clone a repository that includes the above DockerFile.  
The DockerFile is typically in the top level project directory

Open a `host` terminal:
```bash
# Switch to the webapp directory
cd /src/webapp
# Clone the dev repository
git clone --recursive https://github.com/MLHale/nebraska-gencyber-dev-env.git
```

### Step 2: Examine the included DockerFile

In the previous `host` terminal:
```bash
# Switch to the cloned repo directory
cd nebraska-gencyber-dev-env
# Examine the DockerFile
cat Dockerfile
```

### Step 3: Build the image

In the previous `host` terminal:
```bash
# Use the `build` command and supply a DockerFile
# `-t` option provides a name and tag for the image
docker build -t django:dev .
# List local images
docker images
```
> If the build is successful,  `django` appears in your local image listing

## Cleanup

Here is how to delete the image we just created.  
In the previous `host` terminal:
```bash
# Use the `rmi` command and supply a image name
docker rmi django:dev
# List local images
docker images
```
> If the command is successful,  `django` is removed from your local image listing

**Tip**: To delete a container, use the command `docker rm <container-ID>`

---

## Automating Multi-Container Application Build

Apps may require additional services in their environment. For example a Database Service

The `docker-compose` tool automates building your app’s services all at once
and links them as described in a `docker-compose.yml` file

## Django with Postgres Database

`docker-compose.yml` file:

```bash
# Compose file format version
version: "3"
# Declare services
services:
  # Name of the Postgres Database service
  db:
    # Behavior upon container exit
    restart: always
    # Base image is postgres
    image: postgres
    # Per-service volume list
    volumes:
      - postgres-config:/etc/postgresql
      - postgres-data:/var/lib/postgresql/data
      - postgres-logs:/var/log/postgresql
      - ./database-backup:/database-backup
  # Name of the Django service
  django:
    # Use the Dockerfile to build this image
    build: .
    # Overide the default command
    command: python /var/www/backend/manage.py runserver 0.0.0.0:8000
    # Per-service volume list
    volumes:
      - ./backend:/var/www/backend
    # Expose ports
    ports:
     - "8000:8000"
    # Link Django to Postgres
    depends_on:
     - db
# Declare named volumes
volumes:
  # These keys are left empty to use docker engine defaults
  postgres-config:
  postgres-data:
  postgres-logs:
```

## Building with Docker-Compose

The `docker-compose` tool can build containers with a single command.  
In a `host` terminal:
```bash
# Switch to project directory
cd /src/webapp/nebraska-gencyber-dev-env
# build images
docker-compose build
# List local images
docker images
```

> If the build is successful, `nebraskagencyberdevenv_django` and `postgres` appear in your local image listing.
Services are built once and then tagged, by default as `projectname_service`

# Easy Peasy 😏

## Run the App

Before running the built containers, often additional configuration steps are needed.  
The steps below are specific to our setup and will vary with applications

In the previous `host` terminal:
```bash
# run option executes a one-time command against a service
docker-compose run django bash
```

In the returned `container` shell:
```bash
# Perform Django configurations
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser --username admin --email admin
exit
```

Back in the `host` terminal:
```bash
# One simple command to start the entire application
docker-compose up
```

Navigate to http://localhost:8000/ to examine the running app.

# Super cool 🤓

## Shutting down multiple containers

While pressing CRTL+C in the terminal once will shutdown the containers, here is a better way.

In another `host` terminal:
```bash
# Examine running containers
docker ps
# Gracefully shutdown the containers
cd /src/webapp/nebraska-gencyber-dev-env
docker-compose down
```
[Back to TOC](#table-of-contents)

----------


# May the force of containers be with you! 😎
as you take this quiz…
https://www.qzzr.com/c/quiz/430097/the-container-quiz


----------
# Additional Resources

For more information, investigate the following:

- [Container 101](https://www.docker.com/what-container) - What is a container
- [Docker Documentation](https://docs.docker.com) - All things docker
- [Dockerfile](https://docs.docker.com/engine/reference/builder/) - Authoring Reference
- [docker-compose.yml](https://docs.docker.com/compose/compose-file/) - Authoring File Reference
- [Volume](https://docs.docker.com/engine/tutorials/dockervolumes/) - Manage Data in Containers
- [Container Networking](https://docs.docker.com/engine/tutorials/networkingcontainers/#add-containers-to-a-network) - Connect Containers to a Network

[Back to TOC](/doc/Containers-A-primer-UuOqrtvZWkvfqFmWhmdLQ#:uid=751585618020551678160734&h2=Table-of-contents)

----------
# Acknowledgements

This tutorial was initially inspired by [this blog post](https://www.codementor.io/jquacinella/docker-and-docker-compose-for-local-development-and-small-deployments-ph4p434gb) by James. Thanks to thoughtful comments and reviews by [Dr. Matthew L. Hale](http://faculty.ist.unomaha.edu/mhale/)

[Back to TOC](#table-of-contents)


# License
[Nebraska GenCyber](https://github.com/MLHale/nebraska-gencyber) <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

Overall content: Copyright (C) 2017  [Dr. Matthew L. Hale](http://faculty.ist.unomaha.edu/mhale/), [Dr. Robin Gandhi](http://faculty.ist.unomaha.edu/rgandhi/), and [Doug Rausch](http://www.bellevue.edu/about/leadership/faculty/rausch-douglas).

Lesson content: Copyright (C) [Robin Gandhi](http://faculty.ist.unomaha.edu/rgandhi/) 2017.  
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">This lesson</span> is licensed by the author under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
