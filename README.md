# Docker Tutorial Notes

These are some notes I took from a **TechWorld with Nana** [tutorial](https://www.youtube.com/watch?v=3c-iBn73dDE). Maybe it'll help some people out :)

## What is a Container

A way to package applications with all necessary dependencies and configurations. 

These are made up of layers of stacked images. 

The base image might be something like Linux, because of it's small size. 

On top of that, you'd have an application image, followed by configuration data. 

We can pull Docker images straight from Docker's online repository. We can do this with PostgreSQL Database.

```bash
docker run -e POSTGRES_HOST_AUTH_METHOD=trust postgres:9.6
```

If we don't specify version with the `:` it will give us the latest version. We also specify an environmental variable to be set. In our case, we specify that connections can be made to the database without a password (not recommended).

It will search for the image locally. If it doesn't find it, it'll pull from the repo. It then downloads each layer seperately. 

We ran `docker run` rather than `docker pull` which means not only will it pull the image, it will run it, creating a container.

Run the following command to see running containers:

`docker ps`

CLARIFICATION: A Docker Image is the actual package. The Container is when you pull and image on your local machine and actually start it. So if it's not running, it's an image. If you run it, it's a container.

## Docker vs Virtual Machine

An operating system (OS) has two layers: kernel and applications. The kernel interacts with hardware components like CPU. The applications run on the kernel.

Q: So what parts of the OS does Docker Virtualise?
A: Just the Applications layer. It uses the kernel of the host.

A Virtual Machine (VM) has the Applications layer in it's own kernel (so it virtualises the entire OS). It doesn't use the host kernel, it boots up its own.

This results in Docker images being much smaller, whereas VMs can be large. Docker containers can also start much faster. Finally, there's compatibility. With a VM, a VM of any OS can run on any OS host (because it's using it's own Kernel and Application layer).

So if you have a Windows OS, and you want to run a Linux based Docker image. Well, it may not be compatibile. The hosts (windows) kernel may not be compatible with the docker images.

Something like Docker Toolbox helps with this, abstracting away the hosts kernel, allowing it to run different Docker images.

# Docker on Mac

You'll likely want to install the Docker Community Edition from the Docker website.

Before installing, go through system requirements to make sure your mac supports Docker.

If Docker is not compatible, then you'll want to install `Docker Toolbox`. 

Run through [standard installation](https://docs.docker.com/desktop/mac/install/) for Mac. It will provide you with all the stuff you need. Make sure you select whether your Mac's chip is intel or Apple.

## Docker Commands

So, CONTAINER is a running environment for IMAGE. A container contains enviornmental configs, virtual file system, and application image, as well as a port binded to it, so we can talk to the application running inside container. 

To see images on your machine:

```bash
docker images
```

To pull an image from Docker repo (command below pulls latest redis image):

```bash
docker pull redis
```

To start an image in a container. Will pull from repo if not found locally:

```bash
docker run redis
```

To see all running containers:

```bash
docker ps
```

To run container in a detached mode:

```bash
docker run -d redis
```

To restart a container:

```bash
docker stop <CONTAINER ID>
```

To start a container:

```bash
docker start <CONTAINER ID>
```

Show all running container, as well as not running containers:

```bash
docker ps -a
```
To see what logs our container is producing. Good for troublehsooting:

```bash
docker logs <CONTAINER ID>
```

Same as above, but using name of container, rather than ID:

```bash
docker logs <CONTAINER NAME>
```

Create a container, but give it a name. Below we run a redis image, giving it the name `random-name`:

```bash
docker run --name random-name redis
```

Get the terminal of a running container. In other words, enter our container. Below, `it` stands for interactive terminal:

```bash
docker exec -it <CONTAINER ID> /bin/bash
```

Remove a container:

```bash
docker rm <CONTAINER ID>
```

While in your container, you can run standard commands (e.g. `ls`, `pwd`). If you want to exit, just type `exit` and return.

## Port vs Host Port

You'll have noticed if you run two redis containers, without specifying any parameters, they will both be running on the same port (you can see this by running `docker ps`).

So, Containers run on your host machine. This has certain ports available. So a container might be listening on it's own port (5000) and binds to port 5000 on your host machine. 

There would be conflict if you open two 5000 ports on host. However, you could have two containers binded to two different ports on the host machine. 

You can then connect to the container using the port on the host. The host will know how to forward requests to containers using the port binding. 

So if you see something like `5000/tcp` under PORTS when running `docker ps` this means that the container is listening from its own 5000 port, but that we haven't binded a port from our host to the container. This means the container is unreachable. 

So... when we do the `run` command, we have to make sure to create a port binding, like so:

```bash
docker run -p 6000/6379 redis
```

6000 represents our host, and 6379 represents the container.

Rerunning `docker ps` now, you might see something like `6000->6379/tcp` under PORTS.

## Workflow with Docker

Let's say you're working on a JavaScript application using MongoDB. Let's say you want someone else on your team to test it. 

You would add it to a version control system like Git. Then something like Jenkins would produce artifacts, which creates a Docker image. 

This Docker image is pushed to a private Docker Repository. 

A development server then pulls the image from the repo, along with a public MongoDB server (two containers).

## Developing with Containers

Task:

1. Develop UI backend using JavaScript and Node JS. 
2. Use Docker container of a MongoDB database.
3. Deploy Docker container of a Mongo UI (MongoExpress).
4. Create container for application
5. Our browser can then connect to this network of containers

Steps:

1. In Docker Hub, find MongoDB. Pull this:

    ```bash
    docker pull mongo
    ```

2. Pull MongoExpress:

    ```bash
    docker pull mongo-express
    ```

3. Setup a Docker Network

    We need to create an isolated Docker Network, which allows our containers to communicate just by using each other's names.

    Docker automatically generates networks:

    ```bash
    docker network ls
    ```

    We'll create our own network for MongoDB and MongoExpress called `mongo-network`:

    ```bash
    docker network create mongo-network
    ```

4. Run our Mongo container:

    We'll specify the port using `-p` (default for MongoDB is 27017). We'll use this for both our host and container. We'll run it in detached mode using `-d`. We'll also setup some environmetal variables using `-e` to set password and username. Finally, we'll give it a name and specify what network it should be part of using `--name` and `--network` respectively.

    ```bash
    docker run -p 27017:27017 -d -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --network mongo-network mongo
    ```

5. Run MongoExpress:

     The variables we specify can all be found on the Docker Hub for each image. Note we also specify `ME_CONFIG_MONGODB_SERVER=mongodb` which allows it to connect with our MongoDB database:

    ```bash
    docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password --net mongo-network --name mongo-express -e ME_CONFIG_MONGODB_SERVER=mongodb mongo-express   
    ```

6. Enter MongoExpress and create Database:

    Navigate to `http://localhost:8081` and create a database and collection (`user-account` and `userid` respectively)

## Docker Compose

We can use `docker-compose` to do the above in a much simpler way. We basically take all the commands and map them into a .`yaml` file.

An example:

```yaml
services:
    mongodb:
        image:mongo
        ports: 
            -27017:27017
        environment:
            - MONGO_USERNAME=admin
```

Let's map our mongo commandds into a docker compose. It would look something like this:

```yaml
version:'3'
services:
    mongodb:
        image: mongo
        ...
    mongo-express:
        image: mongo-express
        ports: 
            -8080:8080
    environment:
        - ME_CONFIG_MONGODB_A...
        ...
```

Docker compose will take care of creating a common network for both services without us having to explicitly saying this.

REMEMBER: Indentation is really import in YAML files.

To run a docker compose file (docker compose should have been installed when you installed docker desktop):

```bash
docker-compose -f mongo.yaml up
```

`up` will start all containers that are specified in the YAML.

IMPORTANT: When you close and restart a container, all data is lost. There is no data persistance. This can be resolved with volumes (will talk about this later)

To stop containers:

```bash
docker-compose -f mongo.yaml down
```

This will stop all containers and remove the network.

## Building Docker Image

When we want to deploy an application once finished, we want to package it into it's own docker container. So we need to create an image of it.

Let's simulate what something like Jenkins would do to package an application.

### Dockerfile

To build a docker image, we have to copy the contents of an application into a docker file and configue it.

A Dockerfile is a blueprint for creating images.

For example

```Dockerfile
# First line is `FROM image`. So we base our image 
# off another image. This saves us the trouble of 
# installing a bunch of stuff. In the example below 
# we start with node already installed.
# Basically, every image is based on another image.
FROM node

# Alternative to defining environmental images 
# instead of in the docker compose file. Although
# probably better to define thme in docker compose
ENV MONGO_DB_USERNAME=admin

# Execute any linux command. In the below the 
# directory will be created inside the docker
# container.
RUN mkdir -p /home/app

# This executes on the HOST machine. The below will 
# copy files from the current directory on the HOST 
# machine to a folder in the container.
COPY . /home/app

# This executes an entry point linux command. The 
# below will execute `$ node server.js` 
CMD ["node", "/home/app/server.js"]
```

REMEMBER: Dockerfile must be called Dockerfile

To build an image using a Dockerfile:

```bash
docker build -t my-app:version1 .
```

With `-t` we can give our image a tag (usually this would be the version of the image).

The last parameter `.` specifies that the Dockerfile is in our current working directory. 

This will store our image locally. 

We can then test it by creating a container:

```bash
docker run my-app:version1
```

If we want to rebuild an image, make sure containers are stopped, then remove image:

```bash
docker rmi <image id>
```

### Private Repository for Docker Images

Let's look at an AWS service (Elastic Container Registiry or ECR).

To get started, just create a repo and give it a name on AWS.

Here, each image will have it's own image. You can put all your different versions of the same image into the repo.

To push your image into the repo:

1. Login to private repo using docker login

    We don't actually have to use `docker login` for AWS (it gets run in the background). What you'll find in AWS is a command that it'll give you to run that will allow you to connect to your repo. But first, make sure you have AWS Command Line Interface installed, and that it has credentials configured.

2. Once you've authenticated yourself to the repo, we want to push our app to AWS:

    To do this, we need to tag our image. This is basically renaming it to include the AWS repo address. If we don't do this, and we just run `docker push my-app` it'll get pushed to Docker Hub. Instructions will be on AWS for tagging it. The command will start with `docker tag`.

3. Push to AWS:

    `docker push my-app<full tag>`

We can now see this in our AWS repo. 

If we change something in our code, or change our Dockerfile, we'll need to rebuild our image.

```bash
docker build -t my-app.1.1
```

We'll also need to change our tag for when pushing to AWS. Because it's the same image, but just a different version, it should end up in the same AWS repo.

Only layers that have changed will be pushed. Layers that haven't been changed, won't, as they're already there.

AWS ECR can hold around 1000 image versions in a single repo I believe.

Jenkis does all of the stuff we just did on its own.

## Deploy a Dockerised Application

We want to deploy our application on a developement server from our AWS ECR repo. 

So, we'll pull our mongodb stuff from Docker Hub, and pull our application from ECR. 

To do this, we can add our own image (from AWS) to our docker compose yaml file:

```yaml
services:
    my-app:
        image: <our full app tag from aws>
        ports: 3000:3000
    mongdb:
    ...
    mongo-express:
    ...
...
```

To pull this image, our development server needs to login to AWS ECR first. 

So on our development server, we'd run the `docker login <>` command. 

We'd also need our files stored on the development server so we can run:

```bash
docker compose -f <yaml file name> up
```

This is useful, as now anyone can access this development server, run `docker-compose` and test out the application. 

## Docker Volumes

This is for stateful applications. 

Let's say we have a database container, if we stop this container, we will lost all our database changes!

On a host, we have a physical file system. With volumes, we plug or mount this file system into a folder of the virtual filesystem of our Docker container. 

So if something changes on the host file system, it will be seen in the container's file system, and vice versa. 

With `docker run` we'd use the `-v` option to create a volume. There are 3 types of volumes:

**HOST VOLUMES**

```bash
-v /home/mount/data:/var/lib/mysql/data
```

Here we decide where on the host file system that reference is made

**ANONYMOUS VOLUMES**

```bash
-v /var/lib/mysql/data
```

Here Docker automatically creates a folder on the host system

**NAMED VOLUMES** 

```bash
-v name:/var/lib/mysql/data
```

Same as anonymous volumes except we can give a name to the host folder so we can reference the volume by name.

In a production environment we'd probably use the named volumes.

In Docker Compose, things don't change much:

```yaml
services:
    mongodb:
        image: mongo
        volumes:
         - db-data:/var/lib/mysql/data
volumes:
    db-data:
        driver: local
```

Here we list all the volumes we defined at the bottom, and under each service we specify where that volume can be mounted.

This means we can mount the same volume to different containers (this is useful if two containers need to share data)

`local` tells Docker to store the data somewhere locally.

So where are the docker volumes located?

**WINDOWS**: `C:\ProgramData\docker\volumes`
**LINUX**: `/var/lib/docker/volumes`
**MAC**: `/var/lib/docker/volumes`

NOTE: On Mac, Docker creates a Linux Virtual Machine and stores all Docker data there! So you won't find `/var/lib/docker/volumes`. 

To access this VM, just run:

```bash
docker run -it --privileged --pid=host justincormack/nsenter1
```

Congrats! You made it to the end :)
