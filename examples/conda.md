---
title: Conda repositories
parent: Examples
nav_order: 3
---

With Artifactory, you can deploy your Conda packages to a Conda repository for sharing software packages within your team.

To deploy your favourite software package using the Artifactory UI, visit the [online platform](https://rdmrepo.icts.kuleuven.be/ui/login/) and navigate to your `CONDA-REPOSITORY`.
Click "deploy" to deploy your `conda-package.tar.bz2` in this Conda repository.

Alternatively you can use `curl -u <USERNAME>:<API-KEY> -T conda-package.tar.bz2 -X PUT  https://rdmrepo.icts.kuleuven.be/artifactory/REPOSITORY-NAME/` 

*e.g.*

`curl -u u0123456:sb8NAZl53g7tUz5IvbWMGRTJXgtk6GKH96ICImcnwhjyFPTj0iZDymKjVWHmLxRHJsaSdPrT8 -T ujson-4.2.0-py39h295c915_0.tar.bz2 -X PUT https://@rdmrepo.icts.kuleuven.be/artifactory/conda-local/linux-64/`

where:

To install this software package, add the following lines to your .condarc file in your home directory and replace the placeholders:

```
channel_alias: https://<USERNAME>:<API-KEY>@rdmrepo.icts.kuleuven.be/artifactory/api/conda/REPOSITORY-NAME/
channels:
  - https://<USERNAME>:<API-KEY>@rdmrepo.icts.kuleuven.be/artifactory/api/conda/REPOSITORY-NAME/
default_channels:
  - https://<USERNAME>:<API-KEY>@rdmrepo.icts.kuleuven.be/artifactory/api/conda/REPOSITORY-NAME/
```
Now you can activate your local Conda environment and use `conda install <conda-package>` to install your Conda package inside your local Conda environment.
  
  
