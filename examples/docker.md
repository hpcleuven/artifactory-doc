---
title: Docker registries
parent: Examples
nav_order: 2
---

With Artifactory, you can access your own Docker registry for storing and
updating your Docker images and sharing them within your research group.

The following example assumes that you have Docker installed on your local
computer and have built (or pulled) at least one Docker image, which will
here be named `myimage`:
```
docker image ls
```
```
REPOSITORY                                            TAG        IMAGE ID       CREATED        SIZE
myimage                                               latest     ee7033aa2ab2   1 week ago   80.3MB
```

Before uploading to Artifactory, you will need to add tags referring to the
registry as well as the version of your image, e.g.:
```
docker tag myimage registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:latest
docker tag myimage registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:1.0
```
Where \<NAMESPACE\> could be for example `dtai` (Declaratieve Talen en Artificiële Intelligenti)

To upload the tagged images to Artifactory:
```
docker login registry.rdmrepo.icts.kuleuven.be -u <USERNAME> -p <API-KEY>

docker push registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:latest
docker push registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:1.0
# or:
docker push --all-tags registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage
```
To download a specific version:
```
docker pull registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:1.0
```

### Version control

It is essential to assign different tags to different versions of your image,
because Artifactory by itself does not provide version control.

If you would only use the default `latest` tag, then uploading a new image
to Artifactory will effectively overwrite the prior `latest` image in
Artifactory. By adding more specific "release" tags (e.g. `1.0`, `1.1`, ...)
the `latest` image will get overwritten upon uploading (which is fine)
while you retain access to the images with the "release" tags.

In addition to the above, you can apply more advanced version control by:
* always building from a [Dockerfile](
  https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
  (instead of building interactively),
* including the corresponding Dockerfile in a git repository,
* adding the (short) commit hash as a tag when building new images.

To this end, you may want to use a Makefile of the following kind:
```make
REGISTRY := registry.rdmrepo.icts.kuleuven.be
IMAGE := ${REGISTRY}/<NAMESPACE>/myimage
HASH := $$(git rev-parse --short HEAD)

build:
	@docker build -t ${IMAGE}:${HASH} .
	@docker tag ${IMAGE}:${HASH} ${IMAGE}:latest

release: build
	@git tag ${VERSION}
	@docker tag ${IMAGE}:${HASH} ${IMAGE}:${VERSION}

login:
	@docker login ${REGISTRY}

push:
	@docker push --all-tags ${IMAGE}

help:
	@echo 'Example arguments to the make command:'
	@echo '  help                  # prints this help message'
	@echo '  build                 # builds the image and tags it with the last'
	@echo '                        # commit hash and with the "latest" tag'
	@echo '  release VERSION=1.0   # builds the "build" target and adds the'
	@echo '                        # version string as a git tag and image tag.'
	@echo '  login                 # performs a Docker login to our registry'
	@echo '  push                  # pushes all image tags to the registry'
```

After pushing your image to Artifactory with e.g. `make build login push`,
you can then pull specific versions by appending the corresponding commit hash,
```
docker pull registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:ed4f506
```
or the corresponding release tag (if defined via `make release VERSION=...`):
```
docker pull registry.rdmrepo.icts.kuleuven.be/<NAMESPACE>/myimage:1.0
```

> **_NOTE:_** If you are using the KU Leuven Gitlab service for your project,
you can also incorporate image version control in your CI workflow, as described
on the [Gitlab CI](./gitlab_ci) page.


### Dockerhub

When repeatedly pulling the same images from [Dockerhub](
https://hub.docker.com/), there is an advantage in using the Dockerhub mirror
in Artifactory instead. It will act as a local cache, allowing you to
avoid [Dockerhub's rate limitations](https://www.docker.com/increase-rate-limits)
and (hopefully) providing higher download speeds as well.

To e.g. pull a `python:3.8-slim` image from Dockerhub via the mirror:
```
docker pull dockerhub.rdmrepo.icts.kuleuven.be/python:3.8-slim
```


### Containers on HPC with Singularity

Due to the need for superuser privileges, Docker can typically not be used on
HPC clusters. [Singularity](
https://sylabs.io/singularity/) ([VSC documentation](
https://docs.vscentrum.be/en/latest/software/singularity.html)) does not have
this limitation and is available on the login and compute nodes of both Genius
and BrENIAC.

After creating appropriate temporary directories,
```
cd $VSC_SCRATCH
export SINGULARITY_CACHEDIR=$VSC_SCRATCH/singularity_cache
export SINGULARITY_TMPDIR=$VSC_SCRATCH/singularity_tmp
mkdir -p $SINGULARITY_CACHEDIR $SINGULARITY_TMPDIR
```
you can convert your remote Docker image to a local Singularity image (which
will here be named `myimage_latest.sif`):
```
singularity pull --docker-login docker://registry.rdmrepo.icts.kuleuven.be<NAMESPACE>myimage
```
This will again prompt you for a \<USERNAME\> and \<API-KEY\>. You may also specify
these by setting and exporting the corresponding `SINGULARITY_DOCKER_USERNAME`
and `SINGULARITY_DOCKER_PASSWORD` environment variables.

You can now start containers (also in your compute jobs) to execute your
application of choice, e.g.:
```
singularity exec myimage_latest.sif echo "Hello, World"
```

> **_NOTE:_**  Artifactory does not currently support Singularity registries.
  If you want to get Singularity image files into Artifactory, you will need
  to use a [Generic repository](./generic) instead.
