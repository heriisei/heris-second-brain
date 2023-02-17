---
tags: swarm, code, docker, container, server
---

# Swarm

## 1. References
- [Swarm Rocks](https://dockerswarm.rocks/)
- [SwarmPrometheus](https://github.com/stefanprodan/swarmprom)

## 2. Initialization
- Check if the swarm is active by running `docker info`.
- If it is not active, you can activate by running `docker swarm init`.

## 3. CLI
- `docker node ls` -> Check the list of nodes we have.
- `docker service` in a swarm (cluster/multiple host solution) replaces the `docker run` (single host solution).
- `docker service ls` lists all the services in your swarm, also shows the number of replicas that run.
- `docker service ps <service_name>` lists containers of a service. The containers here also will shows in `docker container ls` but with different metadata content.
- `docker service update <service_id> --replicas <number>` -> This will update the replicas number that will run.
- `docker container rm -f <container_name>` ->  This will remove specified container, and if it's a swarm container, then the swarm will respond with creating a new container to fulfill the number of replicas that you've specified before.
	- Re-check the replicas with `docker service ls`.
	- `docker service ps <service_name>` will shows the containers includes the removed one with its state description.
- `docker service rm <service_name>` -> This will remove the service includes the containers.
- `docker swarm join-token manager` -> This will generate a new token to be used when you want to add a new node to join the swarm as a manager node.
- `docker node ps` -> This will lists any containers that run on current host/node/machine. So if you have 3 swarm node, and run a service that replicate one per those node, your current node will only shows one container that runs on the node.
	- `docker node ps <host_name>` -> Lists any containers from specified host name (you can get the host name from `docker node ls`).
	- `docker service ps <service_name>` -> Lists all containers from the service. 
- `watch docker`
	- `watch docker service ls` -> The watch command in front of the docker command will give you a watch mode that will refresh the result within 2 secs.

## 4. Stack
Use docker-compose format, so basically it's just a docker-compose but with additional declarations. So you don't need to manually run the create networks, volumes, services, etc.
- `docker stack deploy -c <compose.yml> <stack_name>` -> To deploy a stack.
- If you've already run a swarm stack from a yaml file, whenever you need to update the running services, just update the yaml, and re-run the `docker stack deploy -c <compose.yml> <stack_name>` to update the running stack, and swarm will automatically detect & update the service.
	- Needs docker-compose version of 3.1 or newer.
	- New `deploy:` key.
	- `docker-compose` ignores `deploy:`, swarm ignores `build:`.
	- [Example File](https://github.com/BretFisher/udemy-docker-mastery/blob/main/swarm-stack-1/example-voting-app-stack.yml)
	- [Udemy course](https://www.udemy.com/course/docker-swarm-mastery/learn/lecture/8681941#overview)
	- [Official Docker Docs](https://docs.docker.com/compose/compose-file/compose-file-v3/#deploy)
- `docker stack ls` -> Lists active stack.
- `docker stack ps <stack_name>` -> Lists tasks that run on the stack, with the node where each of it is active.
	- `docker container ps` or `ls` -> Lists the containers
- `docker stack services <stack_name>` -> Lists the services of the stack, with the replicas data.
- `docker network ls` -> Lists the docker networks, includes the swarm overlay networks.
- `docker stack rm <stack_name>` -> Will remove the stack alongside with it properties like networks, secrets.

### 4.1. Secrets
Encrypted on disk, in transit, and only be available on the place it needs to be. 

#### 4.1.1 Pre-created secrets from CLI
...

#### 4.1.2 Compose file (Simple form)
```yaml
version: "3.1"
services:
	psql:
		...
		secrets:
			- psql_user
			- psql_pw
		environment:
			# Postgres secrets ENV https://hub.docker.com/_/postgres/
			POSTGRES_PASSWORD_FILE: /run/secrets/psql_pw
			POSTGRES_USER_FILE: /run/secrets/psql_user
secrets:
	psql_user:
		file: ./secrets/psql_user.txt
	psql_pw:
		file: ./secrets/psql_password.txt
```
- Then run `docker stack deploy -c compose.yml <stack_name>`.

#### 4.1.3. Compose file (Advanced form)
...

#### 4.1.4. Secrets in the local development
With the same compose file as we used in swarm, we also will able to use the secrets when running it from `docker-compose -f <compose-file> up -d`. Of course, the secrets aren't secured in compose, but if it's only run in development, it will be fine as we need to make the configuration as same as between the development & production.

#### 4.1.5. Custom Secrets Resolver
Sometimes you need to resolve secrets string manually. There is a custom function that can be used to dynamically resolve either it is an env variable or secrets file.

```js
// var.mixin.js

"use strict";

const fs = require("fs");

/**
* Resolve either env vars or docker secrets path to get the real value
* @param {string} file The env var or the file path string
* @returns {string} Return the real string value
*/
module.exports.readVar = (file) => {
	return new Promise(function (resolve, reject) {
		if (file.indexOf("/run/") !== 0) {
			resolve (file);
		} else {
			fs.readFile(`../${file}`, "utf8", function (err, content) {
				if (err) reject(err);
				resolve(content.trim());
			});
		}
	});
};
```
```js
// users.service.js
const { readVar } = require("../mixins/var.mixin");

module.exports = {
	...
	async started() {
		// Create variables to dynamically store resolved value either
		// from an env or secret file.
		const baseUrl = await readVar(process.env.BASE_URL);
		...
	}
	...
}
```

## 5. Full-app lifecycle with compose
We can use this configurations:
- Local `docker-compose up` for development environment
- Remote `docker-compose up` for CI environment
- Remote `docker stack deploy` for production environment
The compose structure may be like this:
 - `docker-compose.yml` -> the base file, which consists of config that will be used in every environment stage.
 - `docker-compose.override.yml` -> by default will be override the base file when we run `docker-compose up -d` in the local development environment.
 - `docker-compose.test.yml` -> freely to rename this one, could be run for test stage with `docker-compose -f <base_file> -f <test_file> up -d`.
 - `docker-compose.prod.yml` -> freely to rename this one, could be run for production with `docker-compose -f <base_file> -f <prod_file> config > <final_file>.yml` and will give you output a combined configuration file to be use with `docker stack deploy`.
 - For the complete example files, check [this](https://github.com/BretFisher/udemy-docker-mastery/tree/main/swarm-stack-3).

## 6. Swarm update 
- Example with commands:
	- `docker service update --image myapp:1.2.1 <service_name>` -> Will update the running service with the desired new docker image. If there is any replicas, they will be updated one by one.
	- `docker service scale service_a=8 service_b=6` -> Scale multiple services at one time.
	- `docker service update --env-add NODE_ENV=production --publish-rm 8080` -> Will add an env variable and remove a published port. `--publish-add 9090:80`
	- `docker service update --force web` -> This will force the whole service nodes to be re-balanced, i.e. if you scale your service but then the load isn't balanced between the old and the new nodes, you can use this command to re-balance the load.  
- Example with stack/compose file:
	- `docker stack deploy -c <file>.yml <stack_name>`

## 7. Node Virtual Host
- Change the hostname on Ubuntu 20.04 LTS, [reference](https://github.com/tiangolo/dockerswarm.rocks/issues/67)
	- `sudo hostnamectl set-hostname $USE_HOSTNAME`
	- `hostname`

## 8. Visualizer Service
`docker service create --name=viz --publish=8080:8080/tcp --constraint=node.role==manager --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock bretfisher/visualizer`
