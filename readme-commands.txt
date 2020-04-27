###On windows
$docker-machine ls

$docker-machine start

$docker-machine env default

### images command
$docker images ls -a


###Container commands
$docker container ls
$docker container run --publish 80:80 --detach nginx				# Run in background
$docker container logs <container id>								#
$docker container top <contianer id>								# Top processes in a container
$docker container rm <container id>
$docker container inspect <container id>							# Inspect a container image configuration data
$docker container stats <container id>								# memory, cpu utilization
$docker container run -it  nginx bash								# run a new container interactively
$docker container exec -it <container id> command					# Run additional command in existing running container

##Some containers
$docker container run --publish 8080:80 --detach --name httpd --network webnetwork httpd
$docker container run --publish 80:80 --detach --name nginx --network webnetwork nginx
$docker container run --publish 3306:3306 --detach --env MYSQL_ROOT_PASSWORD=password --name mysql --network webnetwork mysql

#connect to mysql in container
$docker container exec -it <mysql container id> bash

###Network command
$docker container port <container id>								# show open ports
$docker network ls													# show available network
$docker network inspect <network id>								# show network details like subnet,gateway, attached containers
$docker network create aNewNetwork
$docker network connect  <network name> <container name>			#Connect an running container to another network	
$docker contianer run --network network-name nginx					#run container with a nework	
$docker network disconnect network-name container-name				#disconnect network from a container	
$docker container logs -f ngnix										#watch logs- tail
	
### host vs bridge network
In the case of “bridge“, there is a need for “port mapping“. If you don't specify any network, docker use default bridge network.
In the case of “host“, a container can directly use the host's networking. For example you don't need to open port 80 on nginx

Two containers in non default network can find each other with their name. It happens because DNS server is build in non default networks. 
DNS server is not build by default in bridge network and hence you have to use --link <container name> option when launching containers in bridge network.

	
###DNS round robin (Net alias to run multiple containers with same alias)
$docker container run -detach --network dude --net-alias search --name instance1 elasticsearch:2					#DNS net-alias is same 'search'
$docker container run -detach --network dude --net-alias search --name instance2 elasticsearch:2
$docker container run --rm --net dude alpline nslookup search
$docker container run --rm --net dude centos curl -s search:9200


###Install software e.g. curl
#on ubantu
$apt-get update	   		
$apt-get install -y curl
#on centor
yum update curl


Images	
Images	docker image history <image name>
docker image inspect nginx
docker pull nginx:version

Tags to an image and push	$docker image tag sourceimage[tag]  targetimage[tag]
$docker image tag nginx:latest sethiyasunil/nginx:02092018

$docker login
$docker push sethiyasunil/nginx
DockerFile	$docker image build -t <image name>:<tag name> <directory>
	
	
Volume ( external storage e.g. store database files/date outside container)	
named volume (-v)	docker container run -d --name mysql -e MYSQL_ALLOW_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
volumes created by container	docker volume ls
	
Bind mounth	
	
Docker compose	
setup volumes/network and start all containers	docker-compose up
stop all volumes/network and start all containers	docker-compose down
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	








