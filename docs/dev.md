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
