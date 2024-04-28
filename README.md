https://www.notion.so/Docker-basic-lab-1c45cff034084cdd8d4ff2ea8cb8e310?pvs=4

# Docker basic lab

Before going to Docker Basic, let me briefly overview VM vs. container.

1. VM
    1. Applications are hosted on the related guest OS as virtual machines. 
    2. The image of the VM needs to run no  t only the application's lib, env but also the whole VM config.
2. Container 
    1. Applications are deployed directly on container engines (eg Docker or Podman) by using OS resources. 
    2. It needs to run only the application's source code, lib, and env, so the image size is lightweight and easy to pack, start, and restart. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/12809b32-609c-44a1-86b6-7195fe055f8d/71dbea51-d5d7-45e6-9578-14c0c57beceb/Untitled.png)

Source photo from  NetSolutions

## Docker Flow

there are three-part

1. Client 
    1. Docker file (that is, an important file of Dockerization to build a container based on the image  )
    2. To manage the host from cmd or Docker Desktop 
2. Host
    1. Client command to Docker Deamon of the host via cmd or Docker Desktop to build task based on the information in the Docker file.
3. Registry 
    1. We can share or upload the image with private or public permission to the container registry (eg Docker Hub, ECR).

They connect via API. The brain of a container is the image built from the Docker file.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/12809b32-609c-44a1-86b6-7195fe055f8d/b6c409c0-dd3d-4675-bc37-33f043672d41/Untitled.png)

Source photo from DevKTOps

## Configuration and Testing

Step 1. Create docker file

To test the basics of containerization using Docker, we have a good understanding of the Docker file. 

#will be use Alpine Linux as base image

FROM alpine:latest

#Install nginx [\ means for a sentence]

RUN apk update && \
apk add nginx && \
rm -rf /var/cache/apk/*

#to copy nginx configuration file to default.conf [from … to ]

COPY nginx.conf /etc/nginx/http.d/default.conf

#Creating html directory for hosting

RUN mkdir -p /var/www/html

#Copy index.html to /var/www/html

COPY index.html /var/www/html/

#Change ownership of /var/www/html to user 'nginx'

RUN chown -R nginx:nginx /var/www/html

#Set working directory for hosting

WORKDIR /var/www/html

#Expose port 80 for web service

EXPOSE 80

#Start nginx on container startup

CMD ["nginx", "-g", "daemon off;"]

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/12809b32-609c-44a1-86b6-7195fe055f8d/074b1188-b5b7-4595-8414-e52f57fcb049/Untitled.png)

...

Step 2: Preparing nginx conf for static web site hosting 

server {
listen 80 default_server;
listen [::]:80 default_server;
root /var/www/html;

```
    index index.html index.html;

    # Everything is a 404
    location / {
            try_files $uri $uri/ =404;
    }

    # You may need this to prevent return 404 recursion.
    location = /404.html {
            internal;
    }

```

}

Step 3: Creating HTML file for /var/www/html

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Welcome to DevKTOps</title>
<!-- Bootstrap CSS -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
</head>
<body>
<div class="container">
<div class="jumbotron mt-5">
<h1 class="display-4">Welcome to DevKTOps!</h1>
<p class="lead">This is a simple web page served by nginx inside a Docker container.</p>
<hr class="my-4">
<p>Empowering your DevOps journey.</p>
</div>
</div>
<!-- Bootstrap JS -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
</body>
</html>

Step 4. Building and running the image

#giving the naming

docker build -t hello-kt .

#run the container in the background and map port 8000 on the host to port 80 on the container

docker run --rm -d -p 8000:80 hello-kt
docker run -d -p 8000:80 hello-kt

Step 5. Push the image to the docker hub

#Tag the image

docker tag hello-kt thixpin/hello-kt

#Tag the image with a version

docker tag hello-kt thixpin/hello-kt:1.0

#Docker login

docker login 

#Push the image to Docker Hub

docker push thixpin/hello-kt
docker push thixpin/hello-kt:1.0

**Docker volume and network**

Volume

*To connect one container to another when orchestration

*Data can be lost when deleting a container, so if we create the volume for persistence, we can avoid losing data when deleting the container.

*The volume is mounted on the host, so all reads and writes will be recorded on the host.

*If a new container is used with the same volume, you can continue to use all files from the  deleted container.

Network

*To create virtual network and subnet as LAN to communicate between the containers for files sharing, network call and so on.

#Create a Docker volume

docker volume create my-vol

#List the Docker volumes

docker volume ls

#Create a Docker network

docker network create my-net

#List the Docker networks

docker network ls

#Run the containers with the volume and network

docker run -d --name c1 --network my-net -v my-vol:/usr/share/nginx/html -p 8001:80 nginx:alpine
docker run -d --name c2 --network my-net -v my-vol:/usr/share/nginx/html nginx:alpine

#Add a file to the volume

docker exec -it c1 sh
cd /usr/share/nginx/html
ls
touch hello.html
echo "Hello World" > hello.html
ls

#Check the volume

docker exec c1 ls /usr/share/nginx/html
docker exec c1 ls /usr/share/nginx/html

#List the containers

docker ps -a

#Inspect the Docker volume

docker volume inspect my-vol

#Inspect the Docker network

docker network inspect my-net

#Very connectivty between the containers

docker exec c1 ping c2
docker exec c2 ping c1

#Remove the containers

docker rm -f c1 c2

#Remove the volume

docker volume rm my-vol

#Remove the network

docker network rm my-net

#remove the image

docker rmi hello-kt

Prune the Docker system

#These commands are dangerous and should be used with caution

docker container prune -f # This will remove all the containers
docker volume prune -f # This will remove all the volumes
docker network prune -f # This will remove all the networks
docker system prune -a -f # This will remove all the containers, images, networks, and volumes
