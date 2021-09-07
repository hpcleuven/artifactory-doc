---
title: Docker registries
parent: Examples
nav_order: 2
---

With Artifactory, you can access your own Docker registry for storing and
updating your Docker images and sharing them within your research group.

The following example assumes that you have Docker installed on your local
computer and have build (or pulled) at least one Docker image, which will
here be named `myimage`:
```
docker image ls
```
```
REPOSITORY                                            TAG        IMAGE ID       CREATED        SIZE
myimage                                               latest     ee7033aa2ab2   1 week ago   80.3MB
```

To upload this image to Artifactory:
```
docker login registry.ARTIFACTORY-URL -u USERNAME -p API-KEY
docker tag myimage registry.ARTIFACTORY-URL/GROUPNAME/myimage
docker push registry.ARTIFACTORY-URL/GROUPNAME/myimage
```
and for downloading:
```
docker pull registry.ARTIFACTORY-URL/GROUPNAME/myimage
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
singularity pull --docker-login docker://registry.ARTIFACTORY-URL/GROUPNAME/myimage
```
This will again prompt you for a USERNAME and API-KEY. You can now start
containers (also in your compute jobs) to execute your application of choice,
e.g.:
```
singularity exec myimage_latest.sif echo "Hello, World"
```
