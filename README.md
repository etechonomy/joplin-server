[![Docker Build Status](https://img.shields.io/github/actions/workflow/status/etechonomy/joplin-server/build-image.yml?logo=docker)](https://hub.docker.com/r/etechonomy/joplin-server) ![Docker Pulls](https://img.shields.io/docker/pulls/etechonomy/joplin-server?logo=docker)

---

# Joplin Server

Automated builds of **Joplin Server** in `amd64` and `arm64` &rarr; `docker pull etechonomy/joplin-server`

<img src="JoplinServerIcon.svg" align="left" style="margin-right:15px"/>

**Joplin Server** allows you to sync any Joplin client with it. It includes the ability to share a note with anyone, using a URL. When the note is changed, the content at the URL is changed too. It also features the ability to share a notebook with a user on the same Joplin Server instance. For example, if you share a notebook with another user, that user will see this notebook in their desktop or mobile app, and will be able to edit the notes.

This repository is configured with a GitHub Action that checks for new [Joplin Server](https://joplinapp.org/help/about/changelog/server/) tags every 5 minutes. If a new version is found it will automatically update the tag in this repository and then kickoff another action to build new Joplin Server container images based on the latest tag.

Images can be found here:
https://hub.docker.com/r/etechonomy/joplin-server

---

## Usage

The following table lists the configurable parameters of the `etechonomy/joplin-server` container image:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `APP_BASE_URL` | This is the base public URL where the service will be running. | `http://joplin.yourdomain.tld` |
| `APP_PORT` | The local port on which the Docker container will listen.  | `22300` |
| `DB_CLIENT` | Database client | `pg` |
| `POSTGRES_PASSWORD` |	Postgres DB password | `joplin` |
| `POSTGRES_DATABASE` | Postgres DB database | `joplin` |
| `POSTGRES_USER` | Postgres DB user | `joplin` |
| `POSTGRES_PORT` | Postgres DB port | `5432` |
| `POSTGRES_HOST` | Postgres DB host | `joplin-db` |

> **NOTE:** Follow the [official installation guide](https://github.com/laurent22/joplin/blob/dev/packages/server/README.md) for detailed information.

---

Here are some example `docker-compose` files to help get you started:

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
            - DB_CLIENT=pg
            - POSTGRES_PASSWORD=joplin
            - POSTGRES_DATABASE=joplin
            - POSTGRES_USER=joplin 
            - POSTGRES_PORT=5432 
            - POSTGRES_HOST=joplin-db
        restart: unless-stopped
        ports:
          - 22300:22300
      joplin-db:
        image: postgres:15 # latest as of 4/16/2023
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
- You may need to double check the entrypoint name (`websecure`) and certresolver (`lewildcardresolver`) match your Traefik configuration

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
          - DB_CLIENT=pg
          - POSTGRES_PASSWORD=${JOPLIN_DB_PASS}
          - POSTGRES_DATABASE=joplin
          - POSTGRES_USER=joplin
          - POSTGRES_PORT=5432
          - POSTGRES_HOST=joplin-db
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
          - "traefik.http.routers.joplin-rtr.middlewares=chain-no-auth@file"
          - "traefik.http.routers.joplin-rtr.service=joplin-svc"
          - "traefik.http.services.joplin-svc.loadbalancer.server.port=22300"

      joplin-db:
        image: postgres:15 # latest as of 4/16/2023
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
