---
tags: traefik, proxy, reverse-proxy
---

# Traefik
## Pre-requirements
- Static IP address

## Configuration
- Put these configs for traefik inside the `command` services property at docker-compose, write the yaml tree structure into an object notation connected by dot, e.g: `api.debug=true`. Or, put these as a `traefik.yml` file, and put it inside `volumes` service property at docker-compose, e.g: `- ./traefik/data/traefik.yml:/traefik.yml:ro`.
- Reference:
	- https://doc.traefik.io/traefik/user-guides/docker-compose/acme-dns/

## Dynamic Configuration File

### Access env variable
References: 
- https://community.traefik.io/t/how-to-use-environment-variables-in-the-dynamic-configuration-using-file-provider/4310/2

Steps:
- Define your env variables in respective env file
- Add those variables in `docker-compose.yml` so the container will recognize the variables
- Access those variables in the dynamic configuration file via the syntax below:
	```yml
	http:
		routers:
			pi3one:
				entryPoints:
					- "websecure"
				rule: "Host(`pi3one.{{env "WAN_HOSTNAME"}}`)"
				...
	```