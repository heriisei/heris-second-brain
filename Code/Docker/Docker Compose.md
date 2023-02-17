---
tags: code, docker, server, container
---

# Docker Compose
#docker-compose 

### References
- https://docs.docker.com/compose/reference/up/

We can do layered sets of YAML files, use templates, variables, and more with docker compose. YAML doesn't reads tabs, use only spaces, usually 2 spaces for each indentation. The differences between v2 and v3 are the v2 focus on single-node dev/test and the v3 is focus on multi-node orchestration like swarm or kubernetes.

### CLI
#docker-compose-cli
> Docker Compose CLI (not the `docker-compose.yaml` file) is not designed to be used manually in production

- To create & start the containers of compose, please use:
	- `docker-compose up` -> run the default docker-compose.yml.
	- `docker-compose -f custom-file.yml up` -> run the custom docker-compose file.
	- `docker-compose up -d --build` -> if there are compose services had already running, you can use the `--build` flag to update those services based on the latest/updated version of your codes, without need to run `docker-compose down`.
- To start container/services that have been created before:
	- `docker-compose start` -> run the default docker-compose.yml.
	- `docker-compose -f custom-file.yml -f file-two.yml start` -> run the custom docker-compose file.
- `docker-compose down`
	- `-v` this flag will also remove all the volumes related to the compose, includes the named, don't use this if you want to keep your named volumes.
- `docker-compose exec <service_name> bash` or `sh` -> to get inside the container and interact with bash or sh or any. 
- `docker-compose run <service_name> <service_command>` will create a new container and run the command.
- `docker-compose exec <service_name> sh` will run command from a container that are currently running. 

### Container name
- `container_name: traefik`  is for naming the container. But this is **NOT RECOMMENDED** to use #dont/docker.

### ENV variables
- `COMPOSE_PROJECT_NAME=<name>` -> will sets the prefix container names of your docker-compose services, also another name like for the network names etc, otherwise docker will use your project directory name if this variable is not set, and you better set this out to make the container names clear and not redundant with other projects.

### Image
- To run a service using available prebuild image on the hub:
	```yaml
	services:
		redis: 
			image: 'redis:5.0.8'
	...
	```
- To run a service using custom image from our Dockerfile:
	```yaml
	services:
		users:
			build: '.'
	...
	```

### Command
- The `command` props will override the `CMD` command from the Dockerfile.

### Depends On
The `depends_on` property could be filled with `services` name, not a `container_name`.

### Database URI
#database-uri 
- Database URI will consists of `<dbprovider>://<user>:<password>@<hostname/servicename>:<port>/<dbname>`. Example: `postgresql://root:root@postgres:5432/users` or `postgresql://root:root@tcp(postgres:5432)/users?max_conns=20&max_idle_conns=4`.
- `ports` are optional and consists of pair of two ports, the left side port will published to the host, and the right side port will published to the container. Example: `3307:3306`.

### Persistent Data
#docker-volumes #docker-bind-mounts
- To make docker container reload on change after you modified the code on the host, you could bind the host project directory into the container working directory.
	```yaml
	services:
		users:
			...
			volumes:
				- ./:/app/ 
				# ./host/directory:/container/directory
	```
- There is 3 way to persist data (via docker-compose):
    1. Bind mount to the host directory 
        ```yaml
        services:
        	mysql:
        		volumes:
	        		- ./any/local-directory/you/want:/var/lib/mysql
        ```
    2. Named volume to persist data inside the docker system
        ```yaml
        services:
        	mysql:
        		volumes:
        			- mysqldata:/var/lib/mysql
        volumes:
        	mysqldata:
        ```  
    4. Anonymous volume (treated same as named volume, but with random generated name by docker)
- DON'TS #dont/docker 
	1. Don't use host file paths, if there are any chances that your local development directory structure may be different from others developer machine. The solution is to use dot like this: `.:/container/dir` 
	2. Don't bind-mount databases, bind-mount databases may working fine on Linux machine, but might not work well on Windows or Mac. The solution is to use docker named volume.
	3. Don't use `--build` every time, if you're only run the container for development which usually only need to restart the node not to rebuild the image. 
#### node_modules in Bind-mounts
There are two solutions to make the bind-mounts of `node_modules` work as expected on every OS. Because most containers will run as a Linux, but the host machine can vary, it could be Windows, Mac, or Linux, so the `node_modules` that are installed in the host will be incompatible in the container and vice versa when you do bind-mounts on them.
1. Containerized development: Run `npm install` only from your container, so bind-mounts will also put the node_modules into the host, but you can't do the development directly from your host if your host is not a Linux because the installed `node_modules` has different package architecture.
	1. Run `docker-compose run your_service npm install`, that will install `node_modules` that match with your container architecture which is a Linux.
	2. Run the standard `docker-compose up` with our without flags.
2. Flexible development: This solution will give us two different `node_modules` that enables us to work either from the host or container. 
	1. Edit the Dockerfile:
		```Dockerfile
		FROM node:10.15-alpine
		ENV NODE_ENV=production
		WORKDIR /node # here we will install the container node_modules 
		COPY package*.json ./
		RUN npm install && npm cache clean --force
		WORKDIR /node/app # we move the workdir to give us ability to have separate node_modules
		COPY . .
		CMD ["node", "./bin/www"]
		```
	2. Edit the docker-compose:
		```yaml
		version: '3'
		services:
			express:
				build: .
				ports:
					- 3000:3000
				volumes:
					- .:/node/app #move the container workdir
					- /node/app/node_modules #hide the host node_modules from the container, so it will use node_modules from upper directory (../node)
				environment:
					- DEBUG=sample-express:* 
		```
	
		Explanation: The host could use its own `node_modules` but the container will not use that because it is been hidden by the declared anonymous volume in docker-compose. Then, because there is no `node_modules` are available inside the workdir `/node/app` of the container, the node will go up to the parent directory and find the `node_modules` that are installed in the `/node` directory when the image is built by Dockerfile.  
		
## Dockerhost
#dockerhost

### UFW auto whitelist
- Create `<nameasyouwant.sh>` in the root project
	```bash
	#! /bin/bash
	
	echo "Updating UFW rule"
	CONTAINER_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <yours>_services_dockerhost_1)
	echo "container IP = $CONTAINER_IP"
	yes | sudo ufw delete 1
	sudo ufw allow from $CONTAINER_IP to any port 3306
	```
- Make the file executable via file properties`

## Networks
### Externals:
```yaml
networks:
	<yours>-auth:
	<yours>-public:
		# Allow setting it to false for testing
		external: ${TRAEFIK_PUBLIC_NETWORK_IS_EXTERNAL-true}
```
The above example of external networks declaration, must be provisioned to be created before with `docker network create <yours>-public` command.

## Multiple Compose File
There is a way to make decompose the compose file into separated files to apply multiple stages of environment, like a base compose, dev compose, and prod compose.

### References
- [Advanced Docker Compose Configuration](https://runnable.com/docker/advanced-docker-compose-configuration#:~:text=Using%20Multiple%20Docker%20Compose%20Files,tasks%20against%20a%20Compose%20application.)
- [The Definitive Guide to Docker Compose](https://gabrieltanner.org/blog/docker-compose/#:~:text=docker%2Dcompose%20down-,Using%20Multiple%20Docker%20Compose%20Files,default%2C%20a%20docker%2Dcompose.)
