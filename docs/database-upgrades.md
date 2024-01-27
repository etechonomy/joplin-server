# Upgrading the database in Joplin:

1. Set variable for your data mounts:
	```bash
	DOCKER_VOLUME=/media/docker/volume
	```
	> **NOTE:** This is fairly generic in that my `joplin-data` is mounted at `/media/docker/volume/joplin-data`, so set the variable to wherever you keep your data mounted.

2. Create a backup of the database:
	```bash
	docker exec joplin-db /bin/bash -c '/usr/bin/pg_dump -Fc -U joplin joplin' > ${DOCKER_VOLUME}/joplin-data/backup.dump
	docker kill joplin-db joplin-server
	```

3. At this point, move the mounted directory to some place else. For example:
	```bash
	mv ${DOCKER_VOLUME}/joplin-data ${DOCKER_VOLUME}/joplin-data-pg13
	```

4. Update the Postgres version in the docker-compose.yml file. Example:
	```bash
	joplin-db:
	  image: postgres:16
	```
	> **NOTE:** Reference the sample [docker-compose.server.yml](https://github.com/laurent22/joplin/blob/dev/docker-compose.server.yml) file to find the latest supported database version.
	
5. Run the new image and allow the file structure to be created
	```bash
	docker compose up -d --force-recreate joplin-db joplin-server
	```
	
6. Once the database is up and ready to accept connections (you can check this by viewing the logs), go ahead and copy the `backup.dump` file into the new volume mount, and import the database from the backup you created in step 1.
	```bash
	cp ${DOCKER_VOLUME}/joplin-data-pg15/backup.dump ${DOCKER_VOLUME}/joplin-data/
	docker exec joplin-db pg_restore --clean -U joplin -d joplin /var/lib/postgresql/data/backup.dump
	```
	
7. You're now running the latest supported version of PostgreSQL with Joplin, and all your notes are intact.