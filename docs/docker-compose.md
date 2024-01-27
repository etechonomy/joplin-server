Here are some example `docker-compose` files to help get you started:

### Generic docker-compose.yml

- This is a barebones `docker-compose.yml` example. It is recommended to use a webserver in front of the instance to run it over HTTPS. See the example below using Traefik.

    ```yaml
    ---
    version: '3.7'
    services:
      joplin-server:
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
        image: postgres:16 # latest as of 1/27/2024
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
