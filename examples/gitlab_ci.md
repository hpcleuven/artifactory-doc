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

> **_NOTE:_** This is a basic example -- more advanced setups can be
found at [gitlab.kuleuven.be/gitlab/gitlab-ci](
https://gitlab.kuleuven.be/gitlab/gitlab-ci).

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
    - echo "{\"auths\":{\"${CI_APPLICATION_URL}\":{\"auth\":\"$(echo -n ${CI_APPLICATION_USER}:${CI_APPLICATION_PASSWORD} | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    # Let kaniko build the image and push it to Artifactory:
    - >-
      /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
      --destination "${CI_APPLICATION_URL}/${CI_APPLICATION_NAMESPACE}/${CI_APPLICATION_IMAGE}:${CI_COMMIT_TAG}"
```

You can then securely define the `CI_APPLICATION_...` variables in the
`Settings > CI/CD > Variables` section of the Gitlab project:

* `CI_APPLICATION_URL`: registry.rdmrepo.icts.kuleuven.be
* `CI_APPLICATION_IMAGE`: name of the Docker image
* `CI_APPLICATION_NAMESPACE`: namespace in the Docker registry on Artifactory
* `CI_APPLICATION_USER`: (virtual) Artifactory user (**must be masked!**)
* `CI_APPLICATION_PASSWORD`: API key of the (virtual) user (**must be masked!**)

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
