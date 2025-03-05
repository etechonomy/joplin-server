[![Docker Build Status](https://img.shields.io/github/actions/workflow/status/etechonomy/joplin-server/build-image.yml?logo=docker)](https://hub.docker.com/r/etechonomy/joplin-server) [![GitHub Tag](https://img.shields.io/github/v/tag/etechonomy/joplin-server?logo=joplin&label=latest%20version&labelColor=%23065abd&color=grey)](https://joplinapp.org/help/about/changelog/server/) [![Docker Pulls](https://img.shields.io/docker/pulls/etechonomy/joplin-server?logo=docker)](https://hub.docker.com/r/etechonomy/joplin-server)

:pray: I put a lot of care and effort into maintaining this, and I truly appreciate your support. If you find these container images helpful, please consider supporting my work through one of the following methods: :pray:<br />
<a href="https://www.buymeacoffee.com/etechonomy" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>&emsp;[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/N4N2UGS4G)

---

# Joplin Server

This repo provides automated builds of **Joplin Server** in `amd64` and `arm64` &rarr; `docker pull etechonomy/joplin-server`

<img width=84 src="https://raw.githubusercontent.com/laurent22/joplin/dev/Assets/ImageSources/JoplinServerIcon.svg" align="left" style="margin-right:15px"/>

**Joplin Server** allows you to sync any Joplin client with it. It includes the ability to share a note with anyone, using a URL. When the note is changed, the content at the URL is changed too. It also features the ability to share a notebook with a user on the same Joplin Server instance. For example, if you share a notebook with another user, that user will see this notebook in their desktop or mobile app, and will be able to edit the notes.

<picture><img src="https://raw.githubusercontent.com/etechonomy/joplin-server/main/docs/joplin-server.png"></picture>

---

## Info:

This repository is configured with a GitHub Action that checks for new [Joplin Server](https://joplinapp.org/help/about/changelog/server/) tags every 5 minutes. If a new version is found it will automatically update the tag in this repository and then kickoff another action to build new Joplin Server container images based on the latest tag.

Images can be found here:
https://hub.docker.com/r/etechonomy/joplin-server


---

## Usage

The following table lists the configurable parameters of the `etechonomy/joplin-server` container image:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `APP_BASE_URL` | This is the base public URL where the service will be running. | `http://joplin.yourdomain.tld` |
| `APP_PORT` | The local port on which the container will listen.  | `22300` |
| `DB_CLIENT` | Database client | `pg` |
| `POSTGRES_PASSWORD` |	Postgres DB password | `joplin` |
| `POSTGRES_DATABASE` | Postgres DB database | `joplin` |
| `POSTGRES_USER` | Postgres DB user | `joplin` |
| `POSTGRES_PORT` | Postgres DB port | `5432` |
| `POSTGRES_HOST` | Postgres DB host | `joplin-db` |

### Getting Started:

- [Check here](./docs/docker-compose.md) for example `docker-compose.yml` files
- Follow the [official installation guide](https://github.com/laurent22/joplin/blob/dev/packages/server/README.md) for detailed information.
- Learn to upgrade your database version using my [database upgrade guide](docs/database-upgrades.md).
