---
title: Generic repositories
parent: Examples
nav_order: 1
---

Repositories of the 'generic' type can be used to manage any data objects,
whether binary or text-based. This type is typically used when Artifactory
does not provide a specialized repository for your package type (e.g. plain
executables or movies).

Uploading to and downloading from the repository can be done with the `curl`
command. To create and upload a small text file, execute
```
echo "Hello, World" > test.txt
curl -u USERNAME:API-KEY -T test.txt -X PUT https://rdmrepo.icts.kuleuven.be/artifactory/REPOSITORY-NAME/test.txt
```
You can then download it as follows (e.g. on another location and/or a different
machine):
```
curl -u USERNAME:API-KEY -O https://rdmrepo.icts.kuleuven.be/artifactory/REPOSITORY-NAME/test.txt
```
