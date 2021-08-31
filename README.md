# Joplin Server

Automated builds of **Joplin Server** in amd64, arm64, & arm/v7.

This repository is configured with a GitHub Action that checks for new [Joplin Server](https://github.com/laurent22/joplin/blob/dev/readme/changelog_server.md) tags every 5 minutes. If a new version is found it will automatically update the tag in this repository and then kickoff another action to build new Joplin Server container images based on the latest tag.

Images can be found here:
https://hub.docker.com/r/etechonomy/joplin-server

---

## Developers Notes:

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