---
tags: code, docker, server, container 
---
# Docker
## CLI
#docker-cli
- `docker run --rm imageName:version` -> `--rm` will remove the container after it stopped.
- `docker run -it imageName:version` -> `-it` will start the container in interactive mode (brings up the container shell).
- `docker exec <service_name> bash` or `sh` -> to get inside the container and interact with bash or sh or any. 
- `docker images | grep "REPO\|<yours>"` -> this will be filtering out images that contains `<yours>` name in their `REPOSITORY` column.

### Postgres
- `docker exec -it <container_name> psql -U <db_user> -W <db_name>` 
	-   `docker exec -it` The command to run a command to a running container. The `it` flags open an interactive tty. Basically it will cause to attach to the terminal. If you wanted to open the bash terminal you can do this
	-   `<container_name>` The container name (you could use the container id instead, which in your case would be `40e39bd0329a` )
	-   `psql -U <container_name> -W <db_name>` The command to execute to the running container
	-   `-U` user
	-   `-W` Tell psql that the user needs to be prompted for the password at connection time. _This parameter is optional._ Without this parameter, there is an extra connection attempt which will usually find out that a password is needed, see the [PostgreSQL docs](https://www.postgresql.org/docs/13/app-psql.html).
	-   `<db_name>` the database you want to connect to. There is no need for the `-d` parameter to mark it as the dbname when it is the first non-option argument, see the docs: `-d` "is equivalent to specifying dbname as the first non-option argument on the command line."
	- [Reference](https://stackoverflow.com/a/37100169)
