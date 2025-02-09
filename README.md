# build-definitions

This repository contains components that are installed or managed by the managed CI and Build Team.

This includes default Pipelines and Tasks. You need to have bootstrapped a working appstudio configuration from (see `https://github.com/redhat-appstudio/infra-deployments`) for the dev of pipelines or new tasks.

Pipelines and Tasks are delivered into App Studio via quay organization `redhat-appstudio-tekton-catalog`.
Pipelines are bundled and pushed into repositories prefixed with `pipeline-` and tagged with `$GIT_SHA` (tag will be updated with every change).
Tasks are bundled and pushed into repositories prefixed with `task-` and tagged with `$VERSION` where `VERSION` is the task version (tag is updated when the task file contains any change in the PR)

Currently a set of utilities are bundled with App Studio in `quay.io/redhat-appstudio/appstudio-utils:$GIT_SHA` as a convenience but tasks may be run from different per-task containers.


## Building

Script `hack/build-and-push.sh` creates bundles for pipelines, tasks and create appstudio-utils image. Images are pushed into your quay.io repository. You will need to set `MY_QUAY_USER` to use this feature and be logged into quay.io on your workstation.
Once you run the `hack/build-and-push.sh` all pipelines will come from your bundle instead of from the default installed by gitops into the cluster.

> **Note**
>
> If you're using Mac OS, you need to install [GNU coreutils](https://formulae.brew.sh/formula/coreutils) before running the `hack/build-and-push.sh` script:
> ```bash
> brew install coreutils
> ```

There is an option to push all bundles to a single quay.io repository (this method is used in PR testing). It is used by setting `TEST_REPO_NAME` environment variable. Bundle names are then specified in the container image tag, i.e. `quay.io/<quay-user>/$TEST_REPO_NAME:<bundle-name>-<tag>`

### Pipelines

The pipelines can be found in the `pipelines` directory.

- `core-services`: contains pipeline for the CI of Stonesoup core services e.g. application-service and build-service.
- `template-build`: contains common template used to generate `docker-build`, `fbc-builder`, `java-builder` and `nodejs-builder` pipelines  

### Tasks

The tasks can be found in the `tasks` directories. Tasks are bundled and used by bundled pipelines. Tasks are not stored in the Cluster.
For quick local innerloop style task development, you may install new Tasks in your local namespace manually and create your pipelines as well as the base task image to test new function. Tasks can be installed into local namespace using `oc apply -k tasks/appstudio-utils/util-tasks`.

There is a container which is used to support multiple set of tasks called `quay.io/redhat-appstudio/appstudio-utils:GIT_SHA` , which is a single container which is used by multiple tasks. Tasks may also be in their own container as well however many simple tasks are utilities and will be packaged for app studio in a single container. Tasks can rely on other tasks in the system which are co-packed in a container allowing combined tasks (build-only vs build-deploy) which use the same core implementations.

Shellspec tests can be run by invoking `hack/test-shellspec.sh`.

## Release

Release is done by setting env variable `MY_QUAY_USER=redhat-appstudio-tekton-catalog`, `BUILD_TAG=$(git rev-parse HEAD)` and running `hack/build-and-push.sh`.

### Versioning

When the task update changes the interface (eg. change of parameters, workspaces or results names) then a new version of the task should be created. The folder with the new version must contain `MIGRATION.md` with instructions on how to update the current pipeline file in user's `.tekton` folder.

Adding a new parameter with a default value does not require the task version increase.

Task version increase must be approved by Project Manager.

## Testing

Script `./hack/test-builds.sh` creates pipelines and tasks directly in current namespace and executes there test builds. By setting the environment variable `MY_QUAY_USER` the images will be pushed into user's quay repository, in that case creation of secret named `redhat-appstudio-staginguser-pull-secret` is required.

Script `./hack/test-build.sh` provides way to test on custom git repository and pipeline. Usage example: `./hack/test-build.sh https://github.com/jduimovich/spring-petclinic java-builder`.
