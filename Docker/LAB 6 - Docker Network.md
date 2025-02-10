
## Docker Networking
### Task 1: Create containers and check the connectivity between them.

Launches a CentOS containers named `ct1 and ct2`. By default, they are automatically attached to the default bridge network.
```
docker run -d --name ct1 centos
```
```
docker run -d --name ct2 centos
```
List running containers
```
docker ps
```
List available Docker networks
```
docker network ls
```
Inspect the default bridge network
```
docker network inspect  bridge
```
Access the `ct1` container
```
docker exec -it ct1 bash
```
Check the IP address
```
ip addr
```
Ping `ct2` from `ct1` using its IP address
```
ping <IP address of ct2> -c 5
```
Note : Replace <IP address of ct2> with the actual **IP address** of ct2

```
exit
```
### Task 2: Create a new docker bridge and check connectivity between containers of different bridges
Create a custom bridge network named `ct-bridge1`
```
docker network create ct-bridge1
```
Launches a container in the newly created `ct-bridge1` network
```
docker run --network ct-bridge1 --name ct3 centos
```
Access the `ct3` container
```
docker exec -it ct3 bash
```
Check the IP address
```
ip addr
```
Ping `ct2` from `ct3` using its IP address
```
ping <IP address of ct2> -c 5
```
Note : Replace <IP address of ct2> with the actual **IP address** of ct2
```
exit
```
