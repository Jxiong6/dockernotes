[TOC]

Hi guys, welcome to the chapter of Docker!

First of all, what is Docker?

With Docker, we are able to quickly deploy an application within a few minutes.  So with Docker, we can do simply deployment and avoid error prone manual installations.



## Concepts

### Docker Hub

[Docker hub](https://hub.docker.com/) is a Docker registry  which contains a lot of repositories and different versions of different applications.

### Container

The running version of the image is called a container.

### Docker Architecture



- Docker Client: a client where we run commands, it sends the commands to Docker daemon for execution.
- Docker Daemon: a server component
  - execute the commands
  - manage the containers, local images
  - pull the image from the image repository
  - push the images out to the image repository

For example:

- we run `docker images`
- the Docker Client sends the command to Docker Daemon
- Docker Daemon looks at the local images and sends back the results

### Why Docker is getting more and more popular?

- standardized application packaging
  - Docker provides a consistent env, ensuring that apps and their dependencies can use and be deployed reliably in any env.
  - same packaging for different apps, like python,java, Js
- multi platform support
  - local machine ; data center; cloud(AWS, Azure, GCP)
- light-weight &Isolation
  -  containers are LW
  - container are isolation from each other 
  - VM are heavy-weight



## Common Docker Commands

### Run a container

```bash
# terminate container
Ctrl + C

# when we specify a docker repository, we specify 2 things
<image repository>:<specific version tag>

# run a container
docker run -p 5000:5000 <image repository>:<specific version tag>

# run in detached mode, '-d' is a shortcut for '--detached'
docker run -d -p 5000:5000 <image repository>:<specific version tag>
```

> docker run is the short writing of docker container run 

The great thing about the detached mode is that everything is happening in the background. Each container is assigned an ID when it's launched up

Whenever we run a container, it's part of an internal docker network called a bridge network. By default, all containers run inside the bridge work. You'll not be able to access the container unless the port is exposed outside.

```bash
# -p is a shortcut for --publish
-p <Host Port>:<Container Port>
```

Previously we take the container port 5000 and map it to a host port on the local machine at 5000.

### See docker Logs

```bash
# see the log of a container
docker logs <Container id>

# '-f' means 'follow', follow the logs of a specific application
docker logs -f <Container id>
```

> We don't have to use the entire container id, we can just use a substring like the first four characters.

### Images and containers

```bash
# see all images
docker images

# see all running containers, 'ls' is a shortcut for 'list'
docker container ls

# show all containers, '-a' is a shortcut for '--all'
docker container ls -a

# stop the running container
docker container stop <Container id>
```

```bash
#pull the image, if we dont provide the image tag, it will pull the latest one.
docker pull <image name>
```

```bash
#search for a specific image
docker search <image name>
```

```bash
#view the build history of a docker image
docker image history <image>:<tag>
or
docker image history <image id>
```

```bash
# look at the detail behind the images
docker image inspect <image id>
```

```bash
#remove the image (make sure there are no containers referencing to this image)
docker image remove <image id> or <image name>

#remove the container 
#if the container is running,then we should first stop the container and remove it.
docker container rm <container id>
```

```bash
#pause the container 
docker container pause <container id>
#unpause the container 
docker container unpause <container id>
```



```bash
#kill the container
docker container kill <container id>
```

> difference between `docker kill` and `docker stop`
>
> - docker stop will give the app a chance to close all stuff down 
> - docker kill will stop the container immediately 

```bash
#remove all stopped containers
docker container prune
```

### Docker sysetm

```bash
#show docker disk usage
docker system df

#get real time events from the server
docker system events

#display system-wide infro
docker system infro

#remove unused data(images (no use) and containers(not running) and not used netwoeks and all build cache)
docker system prune -a  

#show all the stats ahout the specific container
docker stats <container id>

# `-m 512m ` assign a specific size of mem usage here 512MB
docker run -d -p 5000:5000 --name=... -m 512m <image repository>:<specific version tag>

#assign a specific cpu% : --cpu-quota=50000   total cpu quota is 100000,we wanna give a half.
docker run -d -p 5000:5000 -m 512m --cpu-quota=50000 <image repository>:<specific version tag>
```

## Docker Project

### Build a image

- First clone the repository.

```
git clone http://github.com/in28minutes/devops-master-class
```

- Navigate to the `project/hello-world/hello-world-python` directory. Now let's build an image for the hello-world -python project.
- now we look at our `dockerfile`

#### Python

```dockerfile
FROM python:alpine3.10   #base image where we are building our app
WORKDIR /app       #working directory for the app
COPY . /app     #copy all the files from the current dirctory to the app directory
RUN pip install -r requirements.txt #download the dependences
EXPOSE 5000 # container port
CMD python ./launch.py # default command to run when a container starts
```

Now we use the dockerfile to build the image.

```bash
# -t means tag
docker build -t <docker hub id - xiongjiahui>/hello-world-python:0.0.2.RELEASE .
```

> The current folder info(whatever infro that is present in the current directory) as the **build context** to build the image. so we need to pass a dot at the end.

Now we do a docker run ,and check it in localhost:5000

```bash
 docker run -p 5000:5000 <docker hub id>/hello-world-python:0.0.2.RELEASE 
```

### Push the image to the Docker Hub

- first sign up for Docker Hub and 

- build 的时候使用

  ```bash
  docker build -t <docker hub id - xiongjiahui>/hello-world-python:0.0.2.RELEASE .
  ```

- push to docker hub 

  - first 

    ```bash
    docker login
    ```

  - then push, make sure that u use the right dockerhub id

    ```bash
    docker push <repository>/hello-world-python:<TAG>	
    ```

  

  ### Making Dockerfile more efficient

  Initial nodejs dockerfile:

  ```dockerfile
  FROM node:8.16.1-alpine
  WORKDIR /app
  COPY . /app
  RUN npm install
  EXPOSE 5000
  CMD node index.js
  ```

  the dependencies dont often change, the code does. If we bulid the dependencies up as a separate layer, there is high chance that the dependency layer also gets cached.

  ```dockerfile
  FROM node:8.16.1-alpine
  WORKDIR /app
  #first copy the dependency as a separate layer (which does not change a lot)
  COPY package.json /app
  RUN npm install
  EXPOSE 5000
  #copy the code which might change
  COPY . /app
  CMD node index.js
  ```

   In the original Dockerfile, using `COPY . /app` copied all files into the container, followed by running `npm install` to install dependencies. This approach caused Docker's cache to invalidate each time any file was modified, necessitating a rerun of `npm install`, even if `package.json` had not changed.
  The improved Dockerfile leverages Docker's layering feature. It first copies the `package.json` file to the `/app` directory and then executes `npm install`. Because `package.json` changes infrequently, Docker effectively caches this layer, ensuring `npm install` is rerun only when `package.json` is modified.

  

  Similarly， we can edit `python`:

  ```dockerfile
  FROM python:alpine3.10
  WORKDIR /app 
  COPY requirements.txt /app/requirements.txt
  RUN pip install -r requirements.txt
  EXPOSE 5000
  COPY . /app
  CMD python ./launch.py
  ```

  ### ENTRYPOINT VS CMD

  with CMD, whatever you pass from the command line wull replace the instructions you wanted to execute. For example if we are using CMD in our Dockerfile and we pass a command after the `docker run`:

  ```bash
  docker run .... ping google.com
  ```

  Then it will run the `ping `command, the docker run command will be replaced.

  However,, entrypoint does not worry about command lime arguments. It wont be overwritten.



## Docker and microservices

We will use 2 microservices and they talk to each other:

- currency conversion service (10USD = 600 INR)(depends on currency exchange service)
- currency exchange service(1 USD =60 INR)

First we 

- Navigato to the `devops-master-class\projects\microservices` directory

The default network mode in Docker is Bridge Network. Containers which are present in the default bridge network are not able to directly talk to each other using `localhost`

```bash
#list the networks of the containers
docker network ls

#inspect the bridge networks
docker network insepct bridge
```

Use these commands we can check the networks of the containers.

To make the 2 services can talk to each other, we create a link between them.

```
docker run -d -p 8000:8000 --name=currency-exchange  in28min/currency-conversion:0.0.1-RELEASE 

docker run -d -p 8100:8100 --env CURRENCY_EXCHANGE_SERVICE_HOST: http://currency-exchange --name=currency-conversion --link currency-exchange in28min/currency-conversion:0.0.1-RELEASE 
```

- first use `link`to create a link between them

- **update the URL** which currency exchange service will be available : http://curency-exchange

- we need to **configure an env variable** with the URL

  ```bash
  --env CURRENCY_EXCHANGE_SERVICE_HOST=http://currency-exchange
  ```

  ```bash
  docker run -d -p 8100:8100 --env CURRENCY_EXCHANGE_SERVICE_HOST= http://currency-exchange --name=currency-conversion --link currency-exchange in28min/currency-conversion:0.0.1-RELEASE 
  ```

The other option is to create a custom network, which means we can create a network for our currency services alone.

```bash
docker network create currency-network

docker run -d -p 8000:8000 --name=currency-exchange  --network=currency-network in28min/currency-conversion:0.0.1-RELEASE 

docker run -d -p 8100:8100  --env CURRENCY_EXCHANGE_SERVICE_HOST= http://currency-exchange --name=currency-conversion --network=currency-network in28min/currency-conversion:0.0.1-RELEASE 

```

> note: the env variable does not change



## Docker compose

As you can see, our command is getting longer.  So we can use a more powerful toll called `docker compose`.It is a tool fro defining and running multi-container docker applications.

Here is the `docker-compose.yml` file

```yml
version: '3.7'
services:
  currency-exchange:
    image: in28min/currency-exchange:0.0.1-RELEASE
    ports:
      - "8000:8000"
    restart: always
    networks:
      - currency-compose-network

  currency-conversion:
    image: in28min/currency-conversion:0.0.1-RELEASE
    ports:
      - "8100:8100"
    restart: always
    environment:
      CURRENCY_EXCHANGE_SERVICE_HOST: http://currency-exchange
    depends_on:
      - currency-exchange
    networks:
      - currency-compose-network
  
# Networks to be created to facilitate communication between containers
networks:
  currency-compose-network:
```

Then we can run the following command 

```bash
docker compose up

# shut down
docker compose down

#see events going on
docker-compose events

#show the configuration
docker-compose config

#show the conatiners
docker-compose ps

#show the tio process of each container
docker-compose top

docker-compose pause
docker-compose unpause

docker-compose stop/kill
```



