---
title: Gitlab CI
parent: Examples
nav_order: 4
---

### In general

Artifactory repositories can be used in continuous integration (CI) workflows
for KU Leuven Gitlab projects ([gitlab.kuleuven.be](
https://gitlab.kuleuven.be)). This allows to e.g. automatically push CI build
artifacts to Artifactory. For more information on Gitlab CI tools, see
[https://docs.gitlab.com/ee/ci](https://docs.gitlab.com/ee/ci).

To enable your CI jobs to push to (or perhaps pull from) your Artifactory
repository, such jobs need similar information as in manual operations:
- the URL of the Artifactory repository,
- a username and API key for authentication.

Since the latter must not be publicly revealed, these have to be provided
by defining them as masked [CI/CD variables to a project](
https://docs.gitlab.com/ee/ci/variables/#add-a-cicd-variable-to-a-project).
We furthermore recommend to not use personal usernames and API keys. Instead,
send us a request to add a virtual 'Gitlab' user to your Artifactory repository
and then use its credentials instead.


### A basic example with Docker

The above applies to any type of CI build artifact. As an illustration,
we will now assume a git repository used for developing a Docker image through
a Dockerfile (see also the page on [Docker](./docker)). We will configure
the CI to build and push a version-tagged image to the Artifactory registry
whenever a new tag is pushed to the `main` branch. For building the image we
choose the `kaniko` tool (see [docs.gitlab.com/ee/ci/docker/using_kaniko.html](
https://docs.gitlab.com/ee/ci/docker/using_kaniko.html)).

> **_NOTE:_** KU Leuven Gitlab repositories can also use the
[Gitlab Container Registry](
https://docs.gitlab.com/ee/user/packages/container_registry) for storing images.

To achieve the above, the `.gitlab-ci.yml` file can look like this:
```yaml
stages:
- build

build:
  stage: build
  rules:
    - if: $CI_COMMIT_TAG && '$CI_COMMIT_BRANCH == "main"'
  image:
    # Select an image for the CI job which has kaniko installed:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    # Create a Docker configuration file holding the authentication info:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(echo -n ${CI_REGISTRY_USER}:${CI_REGISTRY_PASSWORD} | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    # Let kaniko build the image and push it to Artifactory:
    - >-
      /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
      --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
```

You can then securely define the `CI_REGISTRY_...` variables in the
`Settings > CI/CD > Variables` section of the Gitlab project:

* `CI_REGISTRY`: registry.rdmrepo.icts.kuleuven.be
* `CI_REGISTRY_IMAGE`: registry.rdmrepo.icts.kuleuven.be/yournamespace/yourimagename
* `CI_REGISTRY_USER`: (virtual) Artifactory user (**must be masked!**)
* `CI_REGISTRY_PASSWORD`: API key of the (virtual) user (**must be masked!**)

More information about masking Gitlab CI variables can be found on
[docs.gitlab.com/ee/ci/variables/#mask-a-cicd-variable](
https://docs.gitlab.com/ee/ci/variables/#mask-a-cicd-variable).

A CI job will then be triggered after adding a new tag and pushing your
changes, e.g.
```bash
git tag v1.0  # the same tag will also be applied to the image
git push
git push --tags
```
If the job succeeds, a new version of the image will have appeared in
Artifactory, tagged as `v1.0`.


### A more advanced setup with Docker

The above example is a basic one -- more advanced setups can be
found at [gitlab.kuleuven.be/gitlab/gitlab-ci](
https://gitlab.kuleuven.be/gitlab/gitlab-ci). The [Build-Kaniko.gitlab-ci.yml](
https://gitlab.kuleuven.be/gitlab/gitlab-ci/-/blob/master/templates/Jobs/Build-Kaniko.gitlab-ci.yml)
template, for example, will trigger a build if a tag or commit is pushed
to any branch. Build artifacts from different branches will then be pushed
to different "subspaces". The full path to an image for a particular
commit/branch combination will then look like this:
```bash
registry.rdmrepo.icts.kuleuven.be/yournamespace/yourimage/yourbranch:yourcommithash
```
with the `latest` tag pointing to the image of the latest commit of that branch:
```bash
registry.rdmrepo.icts.kuleuven.be/yournamespace/yourimage/yourbranch:latest
```

To use this setup in your CI jobs, the following `.gitlab-ci.yml` suffices:
```yaml
stages:
  - build

include:
  - project: gitlab/gitlab-ci
    ref: master
    file: /templates/Jobs/Build-Kaniko.gitlab-ci.yml
```
This requires the definition of the same four `CI_REGISTRY_...` CI variables
as in the basic example above.
