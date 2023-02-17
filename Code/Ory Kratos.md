---
tags: auth, ory, ory-kratos 
---

# Ory Kratos
## Temporary Setup

References:
- The [Quick-start Documentation](https://www.ory.sh/docs/kratos/quickstart#clone-ory-kratos-and-run-it-in-docker) 

### Steps
- ☑️ Create docker-compose configuration that includes mysql, kratos-migrate, kratos-selfservice-ui, kratos, and mailslurper.
	```yaml
	version: "3.8"
	
	services:
		mysqld:
			image: mysql:5.7
			ports:
				- "33006:3306"
			environment:
				- MYSQL_DATABASE=${DATABASE_ORY_NAME}
				- MYSQL_ROOT_PASSWORD=${DATABASE_ORY_ROOT_PW}
				- MYSQL_USER=${DATABASE_ORY_USER}
				- MYSQL_PASSWORD=${DATABASE_ORY_USER_PW}
			volumes:
				- type: bind
				  source: ./ory/data/mysql
				  target: /var/lib/mysql
			networks:
				- intranet
		# environment variables will automatically create database & user if they're
		# not created yet
	
		kratos-migrate:
			depends_on:
				- mysqld
			image: oryd/kratos:v0.8.0-alpha.3
			environment:
				- DSN=${DATABASE_URL_MYSQL_ORY}
		
		kratos-selfservice-ui-node:
			image: oryd/kratos-selfservice-ui-node:v0.8.0-alpha.3
			ports:
				- "4455:4455"
			environment:
				- PORT=4455
				- SECURITY_MODE=
				- KRATOS_PUBLIC_URL=http://kratos:4433/
				- KRATOS_BROWSER_URL=http://127.0.0.1:4433/
			networks:
				- intranet
			restart: on-failure
		
		kratos:
			depends_on:
				- kratos-migrate
			image: oryd/kratos:v0.8.0-alpha.3
			ports:
				- "4433:4433" # public
				- "4434:4434" # admin
			restart: unless-stopped
			environment:
				- DSN=${DATABASE_URL_MYSQL_ORY}
				- LOG_LEVEL=trace
			command: serve -c /etc/config/kratos/kratos.yml --dev --watch-courier
			volumes:
				- type: bind
				  source: ./ory/kratos/email-password
				  target: /etc/config/kratos
			networks:
				- intranet
		
		mailslurper:
			image: oryd/mailslurper:latest-smtps
			ports:
				- "4436:4436"
				- "4437:4437"
			networks:
				- intranet
	
	networks:
		intranet:
	```
- ☑️ Spin up the docker-compose with `docker-compose -f kratos.yml up`.
- ☑️ Alter the kratos database via MySQL Benchmark or MySQL shell, that change the collation to `utf8mb4_unicode_ci` (make sure the collation name in the “Review SQL Script” window, or use this `ALTER SCHEMA auth DEFAULT COLLATE utf8mb4_unicode_ci;.`
- ☑️ Run database migration by running:
    - `docker exec -it container-kratos sh` — Get into the kratos container shell to run sql migration.
    - `kratos migrate sql -e` ([source](https://github.com/ory/kratos/discussions/2076#discussioncomment-1840543)) — Wait until it says `Successfully applied SQL migrations!`.
- ☑️ After migration, the registration, login, etc. should be works.
- ☑️ You could connect and check the databases from container shell, or from outside the container (e.g. using MySQL Workbench).
- Modify Kratos configuration
	- Change `secrets.cookie`, `secrets.cipher`, etc.
- Find out how to set proper CORS, header request, reverse proxy, authorize API gateway, etc. for better securities

---
## API Client Implementation
### Login
The login flow has *two* consecutive steps, includes the **Initialize Login Flow (Browser)** and **Post Submit Login**. 
### Logout
The logout flow has two consecutive steps, includes the **Create Logout URL (Browser)**  and **Logout** endpoint itself.
### User Account


### To Do
#todo
- [ ] How to set a proper CORS
- [ ] How to set a proper header request
- [ ] Reverse Proxy
- [ ] Authorize the API gateway
