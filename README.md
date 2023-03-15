# Joplin Server

[![Docker Build Status](https://img.shields.io/github/actions/workflow/status/etechonomy/joplin-server/build-image.yml?logo=docker)](https://hub.docker.com/r/etechonomy/joplin-server) ![Docker Pulls](https://img.shields.io/docker/pulls/etechonomy/joplin-server?logo=docker)

---

Automated builds of **Joplin Server** in `amd64`, `arm64`, & `arm/v7`.

This repository is configured with a GitHub Action that checks for new [Joplin Server](https://github.com/laurent22/joplin/blob/dev/readme/changelog_server.md) tags every 5 minutes. If a new version is found it will automatically update the tag in this repository and then kickoff another action to build new Joplin Server container images based on the latest tag.

Images can be found here:
https://hub.docker.com/r/etechonomy/joplin-server

---

## Usage

I would recommend using a frontend webserver to run Joplin over HTTPS.

### Generic docker-compose.yml

- This is a barebones docker-compose example. It is recommended to use a webserver in front of the instance to run it over HTTPS. See the example below using Traefik.

    ```yaml
    ---
    version: '3.7'
    services:
      joplin:
        image: etechonomy/joplin-server:latest
        container_name: joplin-server
        environment:
            - APP_BASE_URL=http://joplin.yourdomain.tld
            - APP_PORT=22300
            - POSTGRES_PASSWORD=joplin
            - POSTGRES_DATABASE=joplin
            - POSTGRES_USER=joplin 
            - POSTGRES_PORT=5432 
            - POSTGRES_HOST=joplin-db
            - DB_CLIENT=pg
        restart: unless-stopped
        ports:
          - 22300:22300
      joplin-db:
        image: postgres:13.3 # latest as of 5/15/2021
        container_name: joplin-db
        restart: unless-stopped
        ports:
          - 5432:5432
        volumes:
          - /foo/bar/joplin-data:/var/lib/postgresql/data
        environment:
          - POSTGRES_PASSWORD=joplin
          - POSTGRES_USER=joplin
          - POSTGRES_DB=joplin
    ```





### Traefik docker-compose.yml

- The following `docker-compose.yml` will make Joplin Server run and apply the labels to expose itself to Traefik.
- Note that there are 2 networks in the example below, one to talk to traefik (`traefik_default`) and one between the Joplin Server and the Database, ensuring that these hosts are not exposed.
- You may need to double check the entrypoint name (`websecure`) and certresolver (`lewildcardresolver`) to match your Traefik configuration

    ```yaml
    ---
    version: "3.7"
    services:
      joplin:
        image: etechonomy/joplin-server:latest
        container_name: joplin-server
        environment:
          - APP_BASE_URL=https://joplin.${FQDN}
          - APP_PORT=22300
          - POSTGRES_PASSWORD=${JOPLIN_DB_PASS}
          - POSTGRES_DATABASE=joplin
          - POSTGRES_USER=joplin
          - POSTGRES_PORT=5432
          - POSTGRES_HOST=joplin-db
          - DB_CLIENT=pg
        # ports:
        #   - 22300:22300
        restart: unless-stopped
        networks:
          - backend
        labels:
          - "traefik.enable=true"
          ## HTTP Routers
          - "traefik.http.routers.joplin-rtr.entrypoints=websecure"
          - "traefik.http.routers.joplin-rtr.rule=Host(`joplin.$FQDN`)"
          - "traefik.http.routers.joplin-rtr.tls=true"
          ## Middlewares
          - "traefik.http.routers.joplin-rtr.middlewares=chain-no-auth@file" # No Authentication
          # - "traefik.http.routers.joplin-rtr.middlewares=chain-basic-auth@file" # Basic Authentication
          # - "traefik.http.routers.joplin-rtr.middlewares=chain-oauth@file" # OAuth 2.0
          ## HTTP Services
          - "traefik.http.routers.joplin-rtr.service=joplin-svc"
          - "traefik.http.services.joplin-svc.loadbalancer.server.port=22300"

      joplin-db:
        image: postgres:13.3 # latest as of 5/15/2021
        container_name: joplin-db
        environment:
        # ports:
        #   - 5432:5432
        volumes:
          - ${VOLUME_DIR}/joplin-data:/var/lib/postgresql/data
        environment:
          - POSTGRES_PASSWORD=${JOPLIN_DB_PASS}
          - POSTGRES_USER=joplin
          - POSTGRES_DB=joplin
        restart: unless-stopped
        networks:
          - backend

    networks:
      backend:
        external:
          name: backend # docker network create --name=backend
    ```

    > **NOTE:** For a more complete view on how to deploy using Traefik, view: https://github.com/etho201/docker-pi-stacks

---
## Developer's Notes:

To create a new tag manually, run:

```bash
printf "Enter a version to tag a release (example: $(git describe --tags --abbrev=0)): " && read -r RELEASE_VER
git tag ${RELEASE_VER}
git push origin ${RELEASE_VER}
```

To delete a tag, run:
```bash
default=$(git describe --tags --abbrev=0) && printf "Enter a mis-tagged release to delete [$default]: " && read -r RELEASE_VER  && : ${RELEASE_VER:=$default}
git tag -d ${RELEASE_VER}
git push origin :${RELEASE_VER}
```
