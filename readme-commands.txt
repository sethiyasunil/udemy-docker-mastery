###On windows
$docker-machine ls


##Install on linux
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl start docker



$docker-machine start

$docker-machine env default

##mount d drive to docker-machine
https://headsigned.com/posts/mounting-docker-volumes-with-docker-toolbox-for-windows/

### images command
$docker images ls -a
$docker search nginx --limit=3				#search top 5 nginx images

###flatten image - It remove all layers of image into a single layer that reduces file disk
# Appoach 1 - export running image into a tar and then import tar to an image.
$docker container run --env MYSQL_ROOT_PASSWORD=password --name mysql mysql
$docker container export mysql >mysqlfatten.tar
$cat mysqlfatten.tar | docker import - mysql:latest


###Image layers understanding
docker info 														# It shows storage driver
$cd /var/lib/docker/overlay2



###docker system commands
$docker system df -v												# utilization by container, image, volume


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
$docker container stop $(docker container ls -aq)					#stop all running containers
$docker container run -d --restart no nginx							# restart policies values : no, on-failure, unless-stopped, always

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
$docker container logs -f ngnix	sleep 500							#default container command is overridden by command "sleep 500"
	
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
#on centos
yum update curl


###Images
$docker image ls 
$docker image history <image name>													# shows all layers of an image
$docker image inspect nginx															# Shows image details - env variable, cmd command, open ports
#docker pull nginx:version															# pull specific image version from docker hub

$docker image tag sourceimage[tag]  targetimage[tag]								#Tags to an image. 
$docker image tag nginx:latest sethiyasunil/nginx:02092018
$docker image build -t <image name>:<tag name> <directory>							#read DockerFile in current folder and build image.

$docker login
$docker push sethiyasunil/nginx

### About Dockerfile
FROM, ENV, RUN, CMD, EXPOSE, COPY, VOLUME, WORKDIR,  LABEL, ADD, ENTRYPOINT, USER
RUN vs CMD : run execute when image build. CMD execute when image container run.
COPY vs ADD : 
COPY: Copy takes in src and destination. It only copy from local file or from directory on your host.
ADD: It also allow you to copy file from	
	a) URL
	b) extract a tar file from the source directly into the destination.
EXPOSE:  It documents port on which contianer listen at runtime. It doesnot actually publish the port.

HEALTHCHECK --interval=5s CMD ping -c 172.17.0.2		
	other otions --interval, --timeout, --start-point, --retries
ENTRYPOINT vs CMD	
		CMD can be over ridden while launching image.
		ENTRYPOINT : It can't be overridden.
		

## create image from running containers
$docker container commit <container name> <image name>
$docker container commit ngnix	mynginx:latest
## You can also change instruction with commit
$docker container commit --change="CMD [bash]" ngnix	mynginx:latest	

### Named Volumes	[external storage outsize container file system (UFS) e.g. store database files/date outside container]
Volumes are not deleted by default.
$docker container run -d --name mysql -e MYSQL_ROOT_PASSWORD=password -v mysql-db:/var/lib/mysql mysql 			#named volume (-v)	
$docker volume ls																								#show volume list


###Bind mounts
Maps a host file or directory to container file or directory. Just two location pointing to  the same file(s).
$docker container run -d --name nginx2 --publish 3306:3306 -v //d/temp:/usr/share/nginx/html nginx				#mapping a host directory/file to a container directory/file
	
### Docker compose	
$docker-compose up/down							#start and stop containers
Compose - Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. ... Define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.
Refer: compose-sample-1

### Swarm
Docker swarm is a container orchestration tool, meaning that it allows the user to manage multiple containers deployed across multiple host machines.
Swarm - Manages group of containers in a cluster. Types of nodes
Manager node - The primary function of manager nodes is to assign tasks to worker nodes in the swarm. Manager nodes also help to carry out some of the managerial tasks needed to operate the swarm. Docker recommends a maximum of seven manager nodes for a swarm
Leader node  -When a cluster is established, the Raft consensus algorithm is used to assign one of them as the "leader node". The leader node makes all of the swarm management and task orchestration decisions for the swarm. If the leader node becomes unavailable due to an outage or failure, a new leader node can be elected using the Raft consensus algorithm.
Worker node - In a docker swarm with numerous hosts, each worker node functions by receiving and executing the tasks that are allocated to it by manager nodes. By default, all manager modes are also worker nodes and are capable of executing tasks when they have the resources available to do so.
	
$docker info   		# check docker swarm is active
$docker swarm init	# initialize a swam
$docker node ls 	
$docker service create alpine ping 8.8.8.8		    #create service 
$docker service ls									#List services
$docker service ps <service_name>					#Lists tasks of a service
$docker service update <id> --replicas 3			##Increments taks count to 3

#Overlay driver - it is swarm wide network driver. 
Use it with --driver overlay while creating network for swarm services.
	
$docker network create --driver overlay mydrupal
$docker service create --name mysql --network mydrupal -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 mysql:5.7
$docker service create --name drupal --network mydrupal -p 80:80 drupal

mysql -u root -p -h 127.0.0.1
	

##Replica va Global Service
Global service create one task on every docker node in cluster.
e.g. $docker service create --name antivirus --mode global norton    			##mode is global and hence swarn run one task on each container No need to sppecify replica


## Lock swarm to protect sensetive information like TLS keys
$docker swarm update --autolock=true
$docker swarm unlock


##draining a docker node
$docker node update --availability drain node2			# docker shutdown containers on node2 and start those tasks on other nodes. Later make it active by --availabilit active

### Routing mesh
Routing Mesh is a feature which make use of Load Balancer concepts.It provides global publish port for a given service.	
	
### volume vs mount
--volume flag was used for standalone containers and the --mount flag was used for swarm services. Consists of multiple key-value pairs, separated by commas.
$docker service create --name mynginx --mount type=volume,source=myvolume, target=/d/mypath nginx
$docker container exec -it <container id> bash


##placemet contstraints
Controls how swarm place containers on nodes
-Replicated or Global services
-Resource constraints [ requirement of CPU or Memory]
-Placement Constraints [ only run on nodes with label pci_compliance =true]
		$docker node update --label-add region=mumbai <container id>
		$docker service create --constraint node.labels.region==mumbai --replicas 3 nginx
-Placement preferences

	
###Stacks	
Stacks is an orchestration on the top of swarm services.
Docker stack is ignoring “build” instructions. You can’t build new images using the stack commands. It need pre-built images to exist. So docker-compose is better suited for development scenarios.	
	
$docker stack deploy -c docker-compose.yml <stack name>	
$docker stack services vote-app				# show services
$docker stack ps vote-app					# show tasks


##Secrets storage
Max size 500 kb

$docker secret create mysql_password file.txt  # bad way to store as file copied on disk
$echo "password" | docker secret create mysql_password -   # again bad way because command history show this password.
$docker secret ls
$docker secret inspect mysql_password
Only container/service to those secret assigned can access secrets
$docker service create --name mysql --secret mysql_password -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql_password  mysql:5.7

Get secret in service
docker container exec -it mysql bash
ls /run/secrets/
ls /run/secrets/mysql_password



### service updates - rolling updates to servics
#add env variable
$docker service update --env-add name=sunil mysql
--env-add key=value # add environment variable
--publish-rm 8080   # remove port
$docker service scale web=5 # it increases replicas to 5

##health check
states: starting, healthy, unhealthy


## run registry
docker container run -d -p 5000:5000 --name registry registry
docker container run -d -p 5000:5000 --name registry -v /d/temp/registry-data:/var/lib/registry registry


######Docker Enterprise Edition

#Universal Control Plane (UCP)
https://success.docker.com/article/install-of-ucp-on-rhel-or-centos-results-in-error-regarding-ports-are-already-in-use-on-your-host


###   UCP Install
sudo docker container run --rm -it --name ucp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/ucp:3.2.6 install \
  --host-address 13.232.194.212 \
  --interactive  --force-minimums
  
  
for i in 179 443 2376 2377 6443 6444 7946 10250 12376 12379 12380 12381 12382 12383 12384 12385 12386 12378 12388; do
  echo adding $i to the firewall
  sudo firewall-cmd --add-port=$i/tcp --permanent
done

#Docker Trusted Registry (DTR)

