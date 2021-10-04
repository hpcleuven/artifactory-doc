---
title: Conda registries
parent: Examples
nav_order: 3
---

With Artifactory, you can deploy your conda packages to a conda repository for sharing software packages within your team.

To deploy your favourite software package using the artifactory UI, visit the online platform and navigate to your `CONDA-REPOSITORY`.
Click "deploy" to deploy your `conda-package.tar.bz2` in this conda repository.

Alternatively you can use `curl -u <USERNAME>:<API-KEY> -X PUT  https://ARTIFACTORY-URL/artifactory/REPOSITORY-NAME/conda-package.tar.bz2` 

To install this software package, add the following lines to your .condarc file in your homedirectory:
```
channel_alias: https://<USERNAME>:<API-KEY>@ARTIFACTORY-URL/artifactory/api/conda/REPOSITORY-NAME/
channels:
  - https://<USERNAME>:<API-KEY>@ARTIFACTORY-URL/artifactory/api/conda/REPOSITORY-NAME/
default_channels:
  - https://<USERNAME>:<API-KEY>@ARTIFACTORY-URL/artifactory/api/conda/REPOSITORY-NAME/
```
Now you can activate your local conda environment and use `conda install <conda-package>` to install your conda package inside your local conda environment.
  
  
