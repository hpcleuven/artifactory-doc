---
title: Conda repositories
parent: Examples
nav_order: 3
---

With Artifactory, you can deploy your Conda packages to a Conda repository for sharing software packages within your team.

To deploy your favourite software package using the Artifactory UI, visit the [online platform](https://rdmrepo.icts.kuleuven.be/ui/login/) and navigate to your <CONDA-REPOSITORY>. 
[image]("https://github.com/hpcleuven/artifactory-doc/blob/main/figures/artifactory-conda_deploy1.png")
Click "deploy" to deploy your `conda-package.tar.bz2` in this Conda repository. A window will pop-up where you can select your conda package. Make sure that the "Target Path" points to either the linux-64 or the noarch subdirectory before deploying the package. 

Alternatively you can use `curl -u <USERNAME>:<API-KEY> -T conda-package.tar.bz2 -X PUT  https://rdmrepo.icts.kuleuven.be/artifactory/<REPOSITORY-NAME>linux-64` 

*e.g.*

`curl -u u0123456:sb8NAZl53g7tUz5IvbWMGRTJXgtk6GKH96ICImcnwhjyFPTj0iZDymKjVWHmLxRHJsaSdPrT8 -T ujson-4.2.0-py39h295c915_0.tar.bz2 -X PUT https://@rdmrepo.icts.kuleuven.be/artifactory/conda-local/linux-64/`

Where the conda <REPOSITORY-NAME> is in this case: `conda-local/`

Conda packages should be deployed in either the linux-64 or the noarch subdirectory of this conda repository. In this case, the ujson package will be deployed in the linux-64 subdirectory.


To install this software package directly from artifactory, add the following lines to your .condarc file in your home directory and replace the placeholders:

```
channel_alias: https://<USERNAME>:<API-KEY>@rdmrepo.icts.kuleuven.be/artifactory/api/conda/<REPOSITORY-NAME>
channels:
  - https://<USERNAME>:<API-KEY>@rdmrepo.icts.kuleuven.be/artifactory/api/conda/<REPOSITORY-NAME>
default_channels:
  - https://<USERNAME>:<API-KEY>@rdmrepo.icts.kuleuven.be/artifactory/api/conda/<REPOSITORY-NAME>
```
Now you can activate your local Conda environment and use `conda install <conda-package>` to install your Conda package inside your local Conda environment.
  
  
