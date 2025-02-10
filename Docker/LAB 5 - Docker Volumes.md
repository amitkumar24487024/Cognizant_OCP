## Volume Mount in Docker
### Task 1: Creating a new docker volume and inspecting containers
Create a new Docker volume named `ct-volume1`
```
docker volume create ct-volume1
```
List all Docker volumes
```
docker volume ls
```
Inspect the newly created volume
```
docker volume inspect ct-volume1
```
Check the volume in Docker's storage directory/Docker Area
```
sudo ls -l  /var/lib/docker/volumes/
```
### Task 2: Launching a Nginx container mapped to the volume
```
docker run -d -p 80:80 --name container1 --mount src=ct-volume1,dst=/usr/share/nginx/html nginx
```
List running containers
```
docker ps
```
Inspect the container to confirm volume mapping
```
docker container inspect container1
```
View the contents of the volume directory on the host
```
sudo ls /var/lib/docker/volumes/ct-volume1/_data/ 
```
Create sample files in the volume directory
```
sudo touch  /var/lib/docker/volumes/ct-volume1/_data/{f1,f2,f3}
```
Create an HTML file in the volume directory
```
sudo vi /var/lib/docker/volumes/ct-volume1/_data/index.html
```
Verify the updated volume content
```
sudo ls -l  /var/lib/docker/volumes/
```
Access the container and verify the volume content
```
docker exec -it container1 bash
```
```
cd /usr/share/nginx/html && ls
```
Exit the container
```
exit
```
## Bind Mount in Docker

### Task 3: Starting Docker Containers Bind Mounts
Create a directory on the host for the bind mount
```
mkdir /home/ubuntu/share
```
Create a sample HTML file in the directory
```
echo 'Hello From Docker Host' > /home/ubuntu/share/index.html
```
Run a container with a bind mount
```
docker run -d -it --name container3 --mount type=bind,source=/home/ubuntu/share/,target=/app nginx:latest
```
Inspect the container to verify the bind mount
```
docker inspect container3
```
Access the container and check the mounted directory
```
docker exec -it container3 bash
```
Inside the container, Confirm that the host's content is accessible in the container
```
cd /app && ls
```
```
cat index.html
```
```
exit
```



