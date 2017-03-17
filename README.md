# CI Deployment Scripts

This repo contains deployment scripts executed during project builds on [Travis CI](https://travis-ci.org)

The scripts should work across a number of repos that include a `Dockerfile`. They are intended to be included as a [Git submodule](https://git-scm.com/docs/git-submodule) within the code repo.

Including this repo within your own (as detailed below) will:
* Build and tag the Docker image as detailed in the repo's `Dockerfile`. This happens on PR creation and update, Tagging (including Release) and commits to the `master` branch
* Push the image to the appropriate docker repo
* If the Travis build has been triggered by a PR a new stack will be created within the `RANCHER_ENVIRONMENT` in Rancher. The rancher and docker templates used to build the stack are pulled from the [nhsuk/nhsuk-rancher-templates](https://github.com/nhsuk/nhsuk-rancher-templates) repo on GitHub. The specific compose files are determined by `RANCHER_TEMPLATE_NAME` and the latest subdirectory (i.e. the highest numbered). This is the (imperfect) convention used by rancher catalogs to implement template versioning.
* Image tag variable names in the rancher-compose.yml file should conform to a naming convention ({modified repo name}_DOCKER_IMAGE_TAG where {modified repo name} is the repo name in upper case with hypens replaced by underscores). This allows the image tag for the current repo being built to be set to PR-_n_ whilst all other image tags are set the default value in rancher compose.
* A publicly accessible URL to the stack will be published in the PR's comments

## Setup in code repo

1. Add this repo as a submodule `git submodule add https://github.com/nhsuk/ci-deployment.git` (note if the submodule is added in a path other than the default the path to the scripts in `.travis.yml` will need updating to that same path)
1. Update that repo's `.travis.yml` to include the sections included within this repo's `.travis.yml` i.e. the ability to run docker and the `after_success` section
1. Setup Travis for the repo. Include all [environment variables](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings) not already available within the Travis environment (described below)
1. Run a build via a PR, TAG, Branch and check everything is working

## Environment variables

As the scripts are intended to be within the Travis CI environment they use a number of environment variables available within that environment by default. The table below highlights those that are used along with additional ones that need to be added to the environment. Additional information about Travis environment variables can be found in the Travis [docs](https://docs.travis-ci.com/user/environment-variables/#Default-Environment-Variables)

| Variable | Secret | Description| Example | Add to Travis? |
|:-----------------------|:-------|:----------------------------------------------------------------------------------------------------------------------------------------------------|:---------------|
| `DOCKER_COMPOSE_VERSION`          | NO     | The  version of the docker CLI to be used | 1.11.1 | YES            |
| `DOCKER_REPO`          | NO     | The docker repo where the built image should be pushed | [nhsuk/profiles](https://hub.docker.com/r/nhsuk/profiles/)                              | YES            |
| `DOCKER_USERNAME`      | NO     | A username that has permissions to push images | | YES            |
| `DOCKER_PASSWORD`      | YES    | The password of the username | | YES            |
| `GITHUB_ACCESS_TOKEN`  | YES    | OAUTH token with [public repo access](https://developer.github.com/v3/oauth/#scopes) | | YES |
| `RANCHER_ACCESS_KEY`   | NO     | The 'user name' part of the API credentials for Rancher access [Rancher API tokens](https://docs.rancher.com/rancher/v1.1/en/api/v1/api-keys/) |     | YES            |
| `RANCHER_SECRET_KEY`   | YES    | The 'password' part of the API credentials for Rancher access [Rancher API tokens](https://docs.rancher.com/rancher/v1.1/en/api/v1/api-keys/) |      | YES            |
| `RANCHER_ENVIRONMENT`  | NO     | The environment within Rancher where the stack will be deployed to | `nhsuk-dev` | YES            |
| `RANCHER_TEMPLATE_NAME`| NO     | Name of directory where Rancher config is stored in [nhsuk-rancher-templates](https://github.com/nhsuk/nhsuk-rancher-templates) | `c2s-profiles` | YES            |
| `RANCHER_TEMPLATE_BRANCH_NAME`| NO     | Optional and for development use only. Gives the ability to specify a branch other than the default value of `master` | `feature\new-template` | OPTIONAL             |
| `RANCHER_URL`          | NO     | The URL of the Rancher environment | https://rancher.nhschoices.net | YES            |
| `SPLUNK_HEC_TOKEN`     | YES    | [HTTP Event Collector token](http://dev.splunk.com/view/event-collector/SP-CAAAE7C)                                                                | | YES            |
| `TRAVIS`             | NO   | Indicates if the environment is `TRAVIS` | | NO             |
| `TRAVIS_PULL_REQUEST`  | NO     | PR number | | NO             |
| `TRAVIS_BRANCH`        | NO     | Branch name triggering the build | | NO             |
| `TRAVIS_TAG`           | NO     | Name of tag triggering the build | | NO             |

There is a [utility script](./configure-travis-env.sh) to set the non-Travis environment variables from the command line.
It assumes you have the Travis CLI installed and you are already logged into Travis.

### Finally

It is worth noting that it is possible to run the PR script locally by setting the environment variables. For example:

 `TRAVIS=true TRAVIS_REPO_SLUG=nhsuk/connecting-to-services TRAVIS_PULL_REQUEST=<PR number of an image which still exists>  RANCHER_TEMPLATE_NAME=c2s-pharmacy-finder SPLUNK_HEC_TOKEN=<splunk token> GITHUB_ACCESS_TOKEN=<personal access token> ./rancher_pr_deploy.sh`

 If testing changes to the rancher template during development  RANCHER_TEMPLATE_BRANCH_NAME can also be set (see above for details).
