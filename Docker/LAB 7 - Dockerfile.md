## Creating containers from Custom Images
### Task 1: Manual - Using commit to build a docker image
Launches an Ubuntu container
```
docker run -d --name webapp-1 ubuntu sleep 5000
```
Access the shell of the webapp-1 container
```
docker exec -it  webapp-1 bash
```
Updates the package list and installs curl, wget, and tree utilities inside the container
```
apt update && apt install curl wget tree -y
```
Verify each installed utilities
```
curl
```
```
wget
```
```
tree
```
```
exit
```
Commit the changes to a new Docker image named `ubuntu:version1`
```
docker commit webapp-1 ubuntu:version1
```
List all available Docker images
```
docker image ls
```
Run a new container named `webapp-2` using the custom image
```
docker run -it --name webapp-2 ubuntu:version1
```
Verify the utilities in the container
```
curl
```
```
wget
```
```
tree
```
CTRL+P+Q

Detach from the container (without stopping it)

### Task 2: Automation - Using Dockerfile to build docker image.
Create a Dockerfile
```
vi Dockerfile
```
Add the given content, by pressing `INSERT`
```Dockerfile
FROM ubuntu
RUN apt update && apt install wget curl tree -y
```
save the file using `ESCAPE + :wq!`

Build the Docker image and tag it as ubuntu:version2
```
docker build -t ubuntu:version2 .   
```
* Syntax :  docker build -t <image-name>:<tag> <path-to-dockerfile>

Verify the created image
```
docker image ls
```
Run a container named `ubuntu-ctr` from the `ubuntu:version2` image
```
docker run -it --name ubuntu-ctr ubuntu:version2
```
Verify the utilities in the container
```
curl
```
```
wget
```
```
tree
```
CTRL+P+Q

Detach from the container (without stopping it)

### Task 3: Building a Dockerfile to setup an Ubuntu container with WordPress application

Create a new directory named `Wordpress` and Navigate into the directory
```
mkdir Wordpress && cd Wordpress
```
Create a Dockerfile
```
vi Dockerfile
```
Add the given content, by pressing `INSERT`

```Dockerfile
FROM ubuntu:20.04
MAINTAINER ADMIN "admin@cloudthat.com"
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
apt-get -q -y install apache2 \
php7.4 \
php7.4-fpm \
php7.4-mysql \
libapache2-mod-php7.4
ADD http://wordpress.org/latest.tar.gz /tmp
RUN tar xzvf /tmp/latest.tar.gz -C /tmp  \
&& cp -R /tmp/wordpress/* /var/www/html
RUN rm /var/www/html/index.html && \
chown -R www-data:www-data /var/www/html
EXPOSE 80
CMD ["/usr/sbin/apache2ctl","-D","FOREGROUND"]

#End of Dockerfile
```
save the file using `ESCAPE + :wq!`

Build the Docker image and tag it as `ct-wordpress:v1`
```
docker build -t ct-wordpress:v1 .
```
Verify the created image
```
docker image ls
```
Create a custom docker bridge network
```
docker network create ct-bridge
```
Run the MySQL container with the necessary environment variables
```
docker run -d --network ct-bridge --name mysql -e MYSQL_DATABASE=wordpress -e MYSQL_USER=admin -e MYSQL_PASSWORD=password -e MYSQL_ROOT_PASSWORD=password mysql:5.7
```
Verify that the MySQL container is running
```
docker ps
```
Run the WordPress container and link it to the MySQL container using the custom bridge network
```
docker run -d --name wordpress --network ct-bridge -p 80:80 ct-wordpress:v1
```
Verify that the WordPress container is running
```
docker ps
```
Access the WordPress container
```
docker exec -it wordpress bash
```
Install `iputils-ping` to test network connectivity
```
apt update && apt install iputils-ping -y
```
```
ping -c 5 mysql
```
After completing the setup, you will be able to access the WordPress site through the browser and begin configuring your WordPress site

Access the WordPress Site:
* Visit ***http://<docker-host-ip>*** in your browser to see your WordPress site’s homepage.
* Enter the admin credentials you set up during installation to access the WordPress dashboard.
  * Database Name: **wordpress** (the name of the database you configured in the MySQL container).
  * Username: **admin** (the user configured for the MySQL database).
  * Password: **password** (the password for the MySQL user).
  * Database Host: **mysql** (the name of the MySQL container, as defined in the Docker network).

*Reference:*
https://docs.docker.com/reference/dockerfile/
