---
tags: code, docker, server, container
---
# Dockerfile
#dockerfile

## Base image
- `FROM` -> please remember to only use the even numbered major/LTS releases, because those are usually a stable release, rather than odd numbered that tends to be an unstable/experimental releases, and also never use the `:latest` tag because it will gives you unexpected updates/breaking changes.
	- The debian is the complete distribution, but it will sized more than 800 MB
	- The alpine distribution is the most popular distribution, sized around 100 MB, focused on security, and recommended to use, but it use alpine package manager (apk) instead of debian package manager (apt)
	- The slim distribution is based on debian distribution, but smaller even than the alpine, not recommended to use since it comes with very minimal packages out of the box that might adds more time and workflow just only to add packages.

## Efficiency
- Each command will be as a different layer, so write it as efficiently as you can. E.g.:
	- Do this
		```Dockerfile
		RUN apt update && apt install
		```
	- Instead of
		```Dockerfile
		RUN apt update
		RUN apt install
		```
	- But on another case, please do this. Separation of two `COPY` commands are means to avoid the `npm install` executed every time you  modified your code (not your package dependencies)
		```Dockerfile
		COPY package.json package-lock.json* ./
		RUN npm install --silent
		COPY . .
		``` 
	- `RUN apt update && apt install curl` avoid to write these debian commands in separate line, write them in a long single line (or new line with `\`) to make sure every time the package install is run, it will install the latest/updated version, in case the `apt update` is cached and never be busted if you put it in the upper line, you might ended up with outdated package registry. 
		- The ampersand `&&` operator do check if the first (left-side) command is successfully run without errors, then it will goes to run the next command (right-side).

## Working directory
- `WORKDIR /<directory>`  this command will create the directory if it doesn't exist.

## Copy files
- `COPY` -> use this to copy files from your host machine into the docker image, `ADD` command can do the same copy but it has many side effects since it actually can do many things other than just copying files, so use `COPY` instead if you're only need to copy files.
	- `COPY file-at-host.ext file-at-image-workdir.ext` the left side is the original file/directory and the right side is the target file/directory that you want to duplicate to.
	- `COPY package.json package-lock.json* ./`  or `package.json package-lock*.json ./` or `package*.json ./`will copy these two files into the current working directory, e.g. `/app`.
		- Take a look at asterisk `*` at the end of the `package-lock.json`, this used as a workaround if the `package-lock.json` is there it will copy the file, and if it is not there the docker will not throw an error.

## Node command
- `CMD` in the last line is the command that will be executed after the image/container is running.
- It is recommended to run a `node` command instead of `npm` here, because with `npm` it will add another layer (the npm) to run the command, while it is not with `node` because it run on the base layer.

## ARG
[Reference](https://docs.docker.com/engine/reference/builder/#arg)
The `ARG` instruction defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag. If a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning. `[Warning] One or more build-args [foo] were not consumed.`

A Dockerfile may include one or more `ARG` instructions. For example, the following is a valid Dockerfile:
```Dockerfile
FROM busybox
ARG user1
ARG buildno
# ...
```

**Warning:**
It is not recommended to use build-time variables for passing secrets like github keys, user credentials etc. Build-time variable values are visible to any user of the image with the `docker history` command.
Refer to the [“build images with BuildKit”](https://docs.docker.com/develop/develop-images/build_enhancements/#new-docker-build-secret-information) section to learn about secure ways to use secrets when building images.

## Multi-stage
#multistage-dockerfile
Write multiple build of images in one dockerfile, and they're could communicate at the build process to use other image as a `FROM` base, to copy files from other image, etc.

Example:
```Dockerfile
FROM node as prod # The first image/stage
ENV NODE_ENV=production
COPY package*.json ./
RUN npm install --only=production && npm cache clean --force
COPY . .
CMD ["node", "./bin/www"]

FROM prod as dev # The second image/stage
ENV NODE_ENV=development
RUN npm install --only=development
CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]

FROM dev as test
ENV NODE_ENV=development
CMD ["npm", "test"]
```

### How to build
- `docker build -t myapp .` -> This standard build command will run through all lines of the above dockerfile, and results in one images which is the `dev` image because that is the final part of the file.
- `docker build -t myapp:prod --target prod .` -> This command will results an image which is the `prod` stage as it declared with the `--target` flag, and named the image with `:prod` tag.
- `docker build -t myapp:test --target=test --no-cache .` -> `--no-cache` will obey the previously cached layer and force to re-run to rebuild the layer.