---
tags: keycloak, auth, sso, 2fa, opensource
---

# 1. Getting Started
## Docker Compose
-  Reference
	- [v17.0.1 eabykov/keycloak-compose](https://github.com/eabykov/keycloak-compose/blob/main/docker-compose.yml)
	- [v16.1.1 official](https://github.com/keycloak/keycloak-containers/blob/main/docker-compose-examples/keycloak-postgres.yml)
## Docker Swarm
You can't use (afaik) postgres volume that were used by keycloak in the docker compose instance. Instead, use a specific volume for the postgres swarm instance. That will lead the keycloak to an error of connection time out: can't connect to db. If you need to use the docker compose data, export the realm then import it into the swarm instance.
# 2. Customization
## 2.1. Make a Custom Image
Some customizations need to be done via editing or overwriting files of the installed keycloak standalone. Because of that, you need to build a new custom image when you run keycloak as a docker container.

### References
- https://robferguson.org/blog/2020/04/12/keycloak-themes-part-1/
- https://github.com/europeana/keycloak-theme/blob/master/Dockerfile

### Steps
1. Create a `Dockerfile`
	```dockerfile
	# Copy image name from docker-compose to be used in here
	FROM quay.io/keycloak/keycloak:16.1.1

	# A modification example of disabling cache on creating a theme
	# Copy a modified standalone-ha.xml into the custom image
	COPY ./keycloak/standalone-ha.xml /opt/jboss/keycloak/standalone/configuration/standalone-ha.xml
	```
	Modification usually works via modify some keycloak files then put it again into the image

2. Modify the `docker-compose.yml`
	```yaml
	...
	services:
		keycloak:
			build:
				context: .
			image: image-name:1.2
			# image: quay.io/keycloak/keycloak:${KC_VERSION}
		...
	```
	Change the `image` properties that directly mentioned the docker image distribution into a new custom image name that will be built from our `Dockerfile` that was referenced via the `build.context` property.

## 2.2. Custom Keycloak Port
There are 3 possible ways to change the Keycloak (docker internal) port. You can just pick one of them depending on what will work on your machine. Because sometimes the way could not be the same on different machines/hosts.

### 2.2.1. Change the port from the keycloak image side
#### Via compose with 2 commands (the most success)
Add these 2 commands into your keycloak service
```yml
...
services:
	keycloak:
		...
		command:
			- -Djboss.http.port=9090
			- -b 0.0.0.0
		...
```

#### Via compose with 1 command
Add these command to your keycloak service
```yml
...
services:
	keycloak:
		...
		command:
			- -Djboss.http.port=9090
		...
```

#### Via standalone-ha.xml
Find and replace the port on this line
```xml
...
<socket-binding-group name="standard-sockets" abcd="abcd">
	...
	<socket-binding name="http" port="${jboss.http.port:8080}"/>
	...
</socket-binding-group>
...
```

### 2.2.2 Change from the compose & traefik sides
```yml
...
services:
	keycloak:
		ports:
			- "9090:9090"
		labels:
			- traefik.http.services.auth-dev.loadbalancer.server.port=9090
	...
```

### 2.3.3 Rebuild the image
`docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --build`

## 2.3. Custom Theme
### References
- https://www.keycloak.org/docs/latest/server_development/#creating-a-theme
- https://robferguson.org/blog/2020/04/12/keycloak-themes-part-1/
- https://www.patternfly.org/v4/guidelines/icons
- https://medium.com/keycloak/change-login-theme-in-keycloak-docker-image-55b5fa5ceec4

### Steps
1. Start the keycloak container (copying files purpose)

2. Copy `standalone-ha.xml` using `docker cp keycloak_container:/opt/jboss/keycloak/standalone/configuration/standalone-ha.xml ./keycloak

3. Disable theme cache so every changes can be seen directly just by refreshing the page (without restarting the keycloak container).
	```xml
	<!-- standalone-ha.xml -->
	...
		<theme>
			<staticMaxAge>-1</staticMaxAge>
			<cacheThemes>false</cacheThemes>
			<cacheTemplates>false</cacheTemplates>
			...
		</theme>
	...
	```

4. Copy default themes from keycloak container into a local directory using `docker cp keycloak_container:/opt/jboss/keycloak/themes/ ./keycloak` .

5. Make a copy of `keycloak/themes/keycloak` theme and give it a new custom name.

6. Change the ownership of the new theme directory from root ownership into your active host user ownership using `sudo chown -R heri keycloak/themes/<yours>`, so you can edit the theme without sudo permission.

7. Stop the currently active/running keycloak container.

8. Edit the `docker-compose.yml` to enable bind-mounts the themes directory.
	```yml
	services:
		keycloak:
			...
			volumes:
				- ./keycloak/themes:/opt/jboss/keycloak/themes/
			...
	```

9. Now you can start again the keycloak container and set a new theme on the realm settings page.

### Custom message
- References:
	- https://www.keycloak.org/docs/latest/server_development/#messages
	  
To change a message text that is displayed on the page, you need to create a `messages` directory inside your theme directory like `keycloak/themes/<yours>/messages/`. Then you can create files for every language you need, for example, if you are using the English language, you can create a `messages_en.properties` file, and you can refer to files inside the `themes/base/messages/` directory for the other languages.

From the above file, you can create new variables or override variables that exist in the base theme.

### Custom page
- References:
	- https://www.keycloak.org/docs/latest/server_development/#creating-a-custom-html-template
	- https://freemarker.apache.org/docs/ref_directive_if.html

To make a custom page/template, you need to copy the files from `themes/base/<theme_type>/`. For example, if you want to make a custom login page, copy `login.ftl` file from `themes/base/login/` and paste it into your `themes/base/<yours>/login/` custom theme directory.

Some variables/properties that are mentioned in the template file are named like the labels on the Keycloak Admin Console page. For example, the label `Login with email` from Realm Settings -> Login page is named as a `realm.loginWithEmailAllowed` property.

### Efficient repository
To minimize the files that are uploaded into your repository, it is safe to only keep any of your theme type and files that will be used. For example, if you only make a custom template for a login page, you could omit any page directory other than `themes/<yours>/login/`. The `themes/<yours>/common/` directory is also safe to omit.

## 2.4. Profile Feature
### References
1. https://www.keycloak.org/docs/16.1/server_admin/#user-profile
2. https://www.keycloak.org/docs/16.1/server_installation/#profiles
3. https://www.marcus-povey.co.uk/2020/10/12/using-the-keycloak-accounts-management-api/
4. https://groups.google.com/g/keycloak-user/c/bEgaeI-t50E/m/q07812frAQAJ
5. https://medium.com/teamarimac/how-to-update-password-with-keycloak-server-8bad1824b944

### About
Profile feature is disabled by default because of it is still in development, but still can be enabled.

### Enabling
- Create a `profile.properties` file and put `profile=preview` in it.
	```env
	profile=preview
	
	# Optional
	feature.account_api=enabled
	```
- Add this as a new line in the `Dockerfile`: `COPY ./keycloak/profile.properties /opt/jboss/keycloak/standalone/configuration/profile.properties`.
- Rebuild the image `docker-compose up --build -d`.
- Enable the feature from Realm Settings -> General page.

### User Attributes
The profile feature will enable User Profile settings (inside the Realm Settings) that allow the administrator to set user attributes. You can use the user attributes to store additional data of every user in keycloak. To configure user attributes, you can follow the instructions on the [1] reference link.

#### Tips
To make a multiple options attributes, you can set a validation rule that has an array of values that contains only the `id` or `reference` of the values, then map the `label` (the text that is displayed) in the Annotation section, easier to do from inside the JSON Editor tab.

Example:
```json
"attributes": [
...
{
  "name": "jobTitle",
  "validations": {
    "options": {
      "options":[
        "sweng",
        "swarch"
      ]
    }
  },
  "annotations": {
    "inputType": "select",
    "inputOptionLabels": {
      "sweng": "Software Engineer",
      "swarch": "Software Architect"
    }
  }
}
...
]
```


### Update Profile
After you set the user attributes, you can let users update their own attributes data via a REST API endpoint. You can follow the instructions on the [3] reference link.

#### Tips
For real-world use cases, you can pass the updated attributes as a raw JSON format like:
```json
{
	"firstName": "Heri",	
	"lastName": "Ris",
	"email": "<yours>@<yours>.com",
	"attributes": {
		"address": [
			"Jl. Medan Merdeka Selatan, Jakarta"
		],
		"major": [
			"144"
		],
		"phone": [
			"0812121212123"
		],
		"degree": [
			"s1"
		],
			"faculty": [
			"009"
		]
	}

}
```
Note that to update the attributes, you need to pass all the writable attributes instead of only passing the modified ones.

### Update Password
To let the users update their password, there is no REST API available because Keycloak removed it due to the security risk concern. One of the available ways is to trigger the keycloak reset password page via the Keycloak client adapter.

For instance, if you're using the JavaScript Keycloak client, then you can call the `keycloak.login()` method with an additional `action: 'UPDATE_PASSWORD'` property, like this:
```js
keycloak.login({
	redirectUri: window.location.href,
	action: 'UPDATE_PASSWORD',
});
```

#### Tips
Don't forget to handle omitting the active session/token if the user chooses to log out from all sessions when they do the reset password flow.

### 2.5. Authentication Scripts Upload Feature
#### References
- https://janikvonrotz.ch/2020/04/30/role-based-access-control-for-multiple-keycloak-clients/
- https://stackoverflow.com/a/59648579

#### Steps
- Add these configuration into the `profile.properties` file
```properties
	...
	feature.scripts=enabled
	feature.upload_scripts=enabled
```

# 3. Export & Import Realm
## References
- https://stackoverflow.com/a/60070937/19253574
- https://keycloak.discourse.group/t/keycloak-17-docker-container-how-to-export-import-realm-import-must-be-done-on-container-startup/13619
- https://github.com/keycloak/keycloak-documentation/blob/main/server_admin/topics/export-import.adoc

## 3.1. Export A Realm (a whole)
### Steps
- Start the auth (keycloak) compose, with pre-requirements:
	- Add a public external network into the keycloak database service
		```yml
		services:
			postgres-keycloak:
				networks:
					- internal-keycloak # The one that connect to keycloak service 
					- external-public # The one that is external network
				...
		networks:
			external-public:
				external: true
		```
- Run this chained commands:
	```bash
	# Run an on/off container that connect to the above external network,
	# with DB envs, and using the keycloak base image that chained with
	# some export commands that will export a json file inside the container
	# /tmp/ directory.
	docker run --rm \
		--name keycloak_exporter \
		--network external-public \
		-e DB_VENDOR=postgres \
		-e DB_DATABASE=<yours_db> \
		-e DB_ADDR=postgres-keycloak:5432 \
		-e DB_USER=<pguser> \
		-e DB_PASSWORD=<pgpassword> \
		quay.io/keycloak/keycloak:16.1.1 \
		-Djboss.socket.binding.port-offset=100 \
		-Dkeycloak.migration.action=export \
		-Dkeycloak.migration.provider=singleFile \
		-Dkeycloak.migration.realmName=<yours-realm> \
		-Dkeycloak.migration.usersExportStrategy=REALM_FILE \
		-Dkeycloak.migration.file=/tmp/<yours-realm-export>.json
	```
- Manually copy the exported file
	- Optionally check the file
		- `docker exec -it keycloak_exporter sh`
		- `cd tmp/`
		- `ls -l`
	- `docker cp keycloak_exporter:/tmp/<yours-realm-export>.json my-local-dir/`
- If the file has been copied, the `keycloak_exporter` could be stopped & removed.

## 3.2. Import A Realm
### Steps (via dashboard)
- Open keycloak dashboard
- click "add realm"
- on the add new realm page, click import instead of directly create an empty realm
- done
### Steps (manual)
- Start the auth (keycloak) compose, with pre-requirements:
	- Add a public external network into the keycloak database service
		```yml
		services:
			postgres-keycloak:
				networks:
					- internal-keycloak # The one that connect to keycloak service 
					- external-public # The one that is external network
				...
		networks:
			external-public:
				external: true
		```
- Run this chained commands
	```bash
	# Run an on/off container that connect to the above external network,
	# with DB envs, and using the keycloak base image that chained with
	# some export commands that will import a json file inside the container
	# /tmp/ directory.
	docker run --rm \
	--name keycloak_importer \
	--network external-public \
	-v /home/user/dir/tothe/exportedfile/tmp:/tmp/keycloak-import \
	-e DB_VENDOR=postgres \
	-e DB_DATABASE=<yours_db> \
	-e DB_ADDR=postgres-keycloak:5432 \
	-e DB_USER=<pguser> \
	-e DB_PASSWORD=<pgpassword> \
	quay.io/keycloak/keycloak:16.1.1 \
	-Djboss.socket.binding.port-offset=100 \
	-Dkeycloak.migration.action=import \
	-Dkeycloak.migration.provider=singleFile \
	-Dkeycloak.migration.strategy=OVERWRITE_EXISTING \
	-Dkeycloak.migration.usersExportStrategy=REALM_FILE \
	-Dkeycloak.migration.file=/tmp/keycloak-import/<yours-realm-export>.json
	```
- If there is no error, the above container will run (not exited), and near the end of the last line, there will be some lines that stated:
	```bash
	KC-SERVICES0030: Full model import requested. Strategy: OVERWRITE_EXISTING
	Full importing from file /tmp/keycloak-import/<yours-realm-export>.json
	Realm '<yours>' imported
	KC-SERVICES0032: Import finished successfully
	```
- If there is errors appear, it might be related to postgres container is not exposed to public yet (check `docker ps -a`), please look back again to the first step.
- You can stop (compose down) the main keycloak & database container (compose).
- Remove the external network (step 1) from the database service.
- Run those main keycloak & database again (compose up).
- The imported realm should be accessible after you login. 
- Or you can import the exported realm via `import` menu in the keycloak sidebar, but I'm not trying this way yet, maybe you need to manually create the realm first for this way.

# 4. API Test with Postman