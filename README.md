# Introduction to Docker

Docker is an open-source platform that allows developers to automate the deployment, scaling, and management of applications using containerization. Containers are lightweight, portable, and encapsulate an application with its dependencies, making it easier to deploy across different environments without worrying about compatibility issues.

This tutorial will walk you through the fundamentals of Docker, its installation, key concepts, commands, and examples of creating and managing Docker containers.

## 1. What is Docker?

Docker allows developers to package applications and their dependencies in a container, ensuring that the application runs consistently across different environments. Unlike virtual machines, which emulate entire operating systems, Docker containers share the host system's OS kernel, making them more efficient in terms of performance and resource usage.

### Benefits of Docker:

**Portability:** Once built, a Docker container can run anywhere that Docker is installed (Windows, Linux, macOS).

**Efficiency:** Containers are lightweight and share the host's OS kernel, making them faster to start and less resource-intensive than VMs.

**Isolation:** Applications are isolated in their containers, reducing the risk of conflicts between different software components.

**Scalability:** Docker makes it easy to scale applications horizontally by adding more containers.

## 2. Key Concepts in Docker

Before diving into using Docker, let’s understand some important concepts:

**Image:** A Docker image is a lightweight, standalone, and executable software package that includes everything needed to run a piece of software: code, runtime, libraries, environment variables, and config files. Images are immutable and form the basis of a container.

**Container:** A container is a runnable instance of a Docker image. Containers are isolated from each other and the host system, with defined limits on resources.

**Dockerfile:** A text file containing instructions to build a Docker image. It specifies how an image should be constructed.

**Docker Hub:** A registry where Docker images can be stored and shared publicly or privately.

**Volume:** A persistent data store that is independent of the container lifecycle. Volumes allow data to persist even after the container is destroyed.

**Network:** Docker networking allows containers to communicate with each other, and with the host system or external networks.

## 3. Docker Installation

### For Windows:

Download Docker Desktop from the official website: [Docker Desktop for Windows](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe).
Run the installer and follow the instructions. Once installed, launch Docker Desktop and ensure it’s running.

### For macOS:
Download Docker Desktop from the official website: [Docker Desktop for Mac](https://desktop.docker.com/mac/main/arm64/Docker.dmg).
Install and open the Docker Desktop application.

### For Linux (Ubuntu):


    # Update your package manager
    sudo apt-get update
    # Install Docker
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    # Start Docker
    sudo systemctl start docker
    # Verify Docker is running
    docker –version


## 4. Working with Docker Images

Docker images are the basis of containers. They include everything needed to run an application: the code, a runtime environment, libraries, environment variables, and configuration files. Here’s an in-depth look at working with Docker images.

### 4.1 Pulling an Image from Docker Hub
Docker Hub is the default registry from which you can pull images, but there are also private and alternative registries.

**Pulling a specific version:** You can specify a tag to pull a specific version of an image.

    docker pull nginx:1.21.1

If no tag is specified, Docker pulls the latest version by default.

    docker pull nginx  # This pulls nginx:latest

Pulling from a different registry:

To pull images from a private registry:

    docker pull myregistry.com/my-image:latest

### 4.2 Listing Images
You can list all downloaded images on your system:

    docker images

Options for docker images:

`-a`: Show all images (including intermediate layers).

`--filter`: Filter images using a condition (e.g., show images created before a certain date).

    docker images --filter "before=nginx:latest"

### 4.3 Building a Custom Image using Dockerfile

A Dockerfile is a text document with all the instructions to assemble an image. Let’s dive into the structure and key instructions of a Dockerfile:

#### Dockerfile Commands

`FROM`: Defines the base image to use (e.g., ubuntu, alpine).

`COPY/ADD`: Adds files from the host system to the container.

`RUN`: Executes a command while building the image (e.g., installing packages).

`CMD`: Defines the default command to run when a container starts.

`EXPOSE`: Declares the port on which the container listens.

`WORKDIR`: Sets the working directory inside the container.

`ENV`: Defines environment variables.

Let’s create a simple Dockerfile:

```
# Use the official Python base image

FROM python:3.9-slim

# Set the working directory

WORKDIR /app

# Copy the current directory contents into the container at /app

COPY . /app

# Install any dependencies

RUN pip install -r requirements.txt

# Run the application

CMD ["python", "app.py"]
```
Build the image:

    docker build -t my-python-app .

### 4.4 Managing Docker Images

**Tagging an Image:** Tagging an image gives it a label, typically in the format name:tag. For example:

    docker tag my-python-app:latest my-python-app:v1.0

**Pushing an Image to a Registry:** After building and tagging your image, you can push it to a registry:

    docker push myrepo/my-python-app:v1.0

**Removing an Image:** You can remove an image from your local system:

    docker rmi my-python-app:v1.0

### 4.5 Image Layers

Every command in a Dockerfile creates a new layer in the image. Docker uses a `Union File System` (UFS) to manage these layers. Layers allow Docker to cache and reuse unchanged layers when building an image, making subsequent builds faster.

**Layer Caching:** Each image layer is cached. If you modify a command early in a Dockerfile (like RUN apt-get update), all subsequent layers will be invalidated and rebuilt. Optimizing the order of commands (e.g., installing dependencies before copying the source code) helps avoid unnecessary rebuilding of layers.

**Understanding Layers:** You can view the layers of an image using:

    docker history <image_id>

Example output:

    IMAGE  CREATED  CREATED BY  SIZE
    
    <image_id>  2 minutes ago  /bin/sh -c apt-get install -y curl  57.6MB
    
    <image_id>  3 minutes ago  /bin/sh -c apt-get update  10.3MB
    
    <image_id>  5 minutes ago  /bin/sh -c FROM ubuntu:20.04  72.9MB

### 4.6 Reducing Image Size

Keeping image sizes small is important for faster deployment, transfer, and start-up times.

**Use Lightweight Base Images:** Start with a minimal base image like alpine or scratch. For example:

    FROM alpine:3.14

**Multi-stage Builds:** This feature helps reduce image size by using separate build and runtime environments. 
Here’s an example:
```
# Stage 1: Build

FROM golang:1.16 AS build-env

WORKDIR /app

COPY . .

RUN go build -o myapp

# Stage 2: Runtime

FROM alpine:latest

WORKDIR /app

COPY --from=build-env /app/myapp /app/myapp

CMD ["/app/myapp"]
```

In this case, the final image contains only the compiled binary, and all build dependencies are discarded.

### 4.7 Image Optimization Best Practices

**Avoid Including Unnecessary Files:** Use .dockerignore to exclude files and directories that aren’t needed in the container (e.g., local test files or .git directories).

    .git
    node_modules
    *.log

**Minimize Layers:** Combine multiple commands into one RUN instruction to reduce the number of layers.

    RUN apt-get update && apt-get install -y \
    curl \
    vim \
    && apt-get clean

## 5.Working with Containers

A container is a running instance of a Docker image. Containers provide an isolated environment for the application.

### 5.1 Running a Container

The docker run command is used to start a container. Here are some additional options:

`-d`: Run the container in detached mode (in the background).

`-it`: Run the container interactively (useful for testing).

`--name`: Assign a name to the container.

`-p`: Map container ports to host ports (e.g., -p 8080:80 maps port 80 of the container to port 8080 on the host).

`-v`: Mount a volume (e.g., -v /host/path:/container/path).

`-e`: Assign environment variable

    docker run -d -p 8080:80 --name webserver nginx

This runs an nginx container in detached mode (-d) and maps port 8080 of the container to port 80 on the host (-p) with container name as webserver.

### 5.2 Managing Containers

#### 5.2.1 Listing Containers

To see all running containers:

    docker ps

For all containers, including stopped ones:

    docker ps -a

#### 5.2.2 Stopping and Removing Containers

To stop a running container:

    docker stop <container_id>

To remove a container:

    docker rm <container_id>

To remove all stopped containers:

    docker container prune

### 5.3 Inspecting Containers

You can inspect the details of a container using:

    docker inspect <container_id>

This command returns detailed information in JSON format, including the configuration, network settings, volumes, etc.

### 5.4 Logging and Debugging

To see the logs of a container:

    docker logs <container_id>

To follow logs in real-time:

    docker logs -f <container_id>

5.5 Container Lifecycle Commands

**Pausing and Unpausing Containers:** You can pause a container's processes (e.g., during backups) and resume them later.

    docker pause <container_id>
    
    docker unpause <container_id>

**Restart Policies:** Containers can be automatically restarted if they crash or are stopped.

`Always` restart the container if it stops.

    docker run --restart always nginx

`On-failure` restart only if the container exits with a non-zero status code.

    docker run --restart on-failure nginx

### 5.5 Container Stats: 
To monitor real-time resource usage (CPU, memory, I/O, etc.) of running containers:

    docker stats

### 5.6 Container Resource Limiting: 
You can limit the resources (CPU and memory) available to a container.

    docker run -d --memory="256m" nginx

Limit CPU usage (in this example, the container can use only half of one CPU core):

    docker run -d --cpus="0.5" nginx

### 5.7 Container Security

Docker containers share the host’s kernel, so security is important to prevent containers from negatively impacting the host system or other containers.

**User Permissions:** Always run applications inside the container as a non-root user when possible. In your Dockerfile, you can create and switch to a non-root user:

**User Permissions:** Always run applications inside the container as a non-root user when possible. In your Dockerfile, you can create and switch to a non-root user:

**Seccomp, AppArmor, and SELinux:** These Linux security features help isolate containers from each other and the host system.

Use Seccomp to restrict the system calls available to containers.

    docker run --security-opt seccomp:unconfined nginx

## 6. Docker Volumes

Docker volumes are used for persisting data generated by Docker containers. By default, all data inside a container is ephemeral (it disappears when the container is removed), but volumes allow you to persist data beyond the container's lifecycle.

### 6.1 Creating a Volume

	docker volume create my-volume

### 6.2 Listing Volumes:

    docker volume ls

### 6.3 Inspecting a Volume: 
Get detailed information about a specific volume:

    docker volume inspect my-volume

### 6.4 Removing a Volume:

    docker volume rm my-volume

### 6.6 To remove all unused volumes:

    docker volume prune

### 6.7 Attaching a Volume to a Container

When running a container, you can attach a volume to a directory in the container using the `-v` or `--mount` option.

    docker run -d -v my-volume:/data nginx

This mounts the `my-volume` to the `/app/data` directory inside the container.

### 6.8 Types of Volume Mounts

Docker supports three types of mounts: volumes, bind mounts, and tmpfs mounts.

#### Volumes (Managed by Docker):

Docker volumes are stored in the host’s filesystem but managed by Docker.

Volumes persist after the container is removed.

Data volumes can be shared between multiple containers.

    docker run -d -v /data nginx

#### Bind Mounts (Direct Mapping to Host):

Bind mounts map a file or directory on the host to a location in the container.

Use bind mounts when you need tight control over the exact location of the file on the host.

    docker run -d -v /host/directory:/container/directory nginx

#### tmpfs Mounts (In-Memory Storage):

tmpfs mounts are stored in the host’s memory, not on disk. These mounts are temporary and are cleared when the container stops.

    docker run --tmpfs /container/tmp:rw,size=64m nginx

6.9 Data Persistence and Sharing

**Named Volumes:** A named volume can be reused across multiple containers, which is useful for sharing data.

    docker volume create my-named-volume
    docker run -v my-named-volume:/data nginx

**Backup and Restore a Volume:** 
To backup the contents of a volume:

    docker run --rm -v my-named-volume:/volume-data -v $(pwd):/backup busybox tar cvf /backup/backup.tar /volume-data

To restore a volume from a backup:

    docker run --rm -v my-named-volume:/volume-data -v $(pwd):/backup busybox tar xvf /backup/backup.tar -C /volume-data

## 7. Docker Networks

Networking in Docker allows containers to communicate with each other and the outside world. Docker sets up different types of networks, which can be customized.

### 7.1 Types of Docker Networks

**Bridge (default):** A bridge network is created by default when you start Docker. Containers on the same bridge network can communicate with each other using their container names. Use when you want to isolate containers and provide selective communication.

    docker network create my-bridge-network

**Host:** The container shares the host machine’s network stack. The container does not get its own IP, but uses the host’s IP address. Removes network isolation between the container and the host.

    docker run --network host nginx

**Overlay:** Used in multi-host setups (like Docker Swarm), overlay networks enable containers to communicate across different Docker hosts.

    docker network create -d overlay my-overlay-network

**None:** This disables networking for the container.

    docker run --network none nginx

### 7.2 Managing Networks

#### Listing Networks:

    docker network ls

#### Inspecting a Network: 
To get detailed information about a specific network:

    docker network inspect my-network

#### Removing a Network:

    docker network rm my-network

### 7.3 Connecting Containers to a Network

You can connect a running container to a Docker network:

    docker network connect my-network my-container

Likewise, to disconnect a container from a network:

    docker network disconnect my-network my-container

### 7.4 DNS and Service Discovery

Docker automatically assigns a DNS name to each container based on its name. Containers can communicate using these names if they are on the same network:

    docker run --network my-network --name container1 nginx
    docker run --network my-network --name container2 nginx

Now, `container1` can reach `container2` by simply using the name `container2`.

### 7.5 Advanced Docker Network Configuration

**Custom DNS Servers:** By default, Docker uses Google DNS servers (8.8.8.8). You can override this using the `--dns` option:

    docker run --dns 1.1.1.1 nginx

**Static IP Addressing:** You can assign a static IP address to a container within a user-defined bridge network.

    docker network create my-net --subnet=192.168.1.0/24
    docker run --net my-net --ip 192.168.1.100 nginx

## 8. Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. A docker-compose.yml file is used to define services, networks, and volumes.

Example docker-compose.yml


    version: '3'
    services:
      web:
        image: nginx
        ports:
          - "80:80"
      db:
        image: mysql
        environment:
          MYSQL_ROOT_PASSWORD: password

To start the application:

    docker-compose up

To stop it:

    docker-compose down

# 9. Best Practices

**Use Small Base Images:** Using minimal base images like alpine can reduce image size.

**Layer Caching:** Docker caches layers, so order commands to leverage this feature. Place frequently changing instructions (like COPY) later in the Dockerfile.

**Use Multi-stage Builds:** For large images, use multi-stage builds to keep the final image lean.

**Environment Variables:** Use environment variables to configure your containers, making them easier to manage in different environments.











































