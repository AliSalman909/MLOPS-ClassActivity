# MLOps Continuous Delivery Demo

A tutorial project for learning how to test, package, and release a small Flask API using Docker and GitHub Actions. The demo prediction doubles the input number; it does not use a trained ML model.

## Progress by commit

The summaries below describe the completed steps; commit titles follow the commands used during the tutorial.

| Commit / step | What we did |
| --- | --- |
| Flask API and dependencies | Added `app.py` with `/`, `/health`, and `/predict` endpoints, plus Flask and pytest dependencies in `requirements.txt`. |
| API tests and requirements fix | Added tests for the health endpoint and prediction result. Removed terminal commands from `requirements.txt`, installed dependencies in a virtual environment, and ran the tests. |
| Docker containerization | Added a Dockerfile using Python 3.12, built the local image, ran the container, and checked `/health`. |
| Delivery principle | Documented our progress and the build-once, deploy-many principle. |
| Application versioning | Recorded the initial application version, `1.0.0`, in `VERSION`. |
| Container registry | Documented GHCR as the planned location for release images. |
| Start the CD workflow | Step 10 defines a tag-triggered workflow that checks out the code, sets up Python 3.12, installs dependencies, and runs the API tests. |
| Extract the release version | Step 11 adds a build job after the tests, removes the leading `v` from the pushed tag, and exposes the version for later steps and jobs. |
| Registry login | Step 12 configures GHCR authentication in the build job using `docker/login-action@v3` and the workflow's automatic `GITHUB_TOKEN`. |
| Build and publish image | Step 13 adds Docker build and push steps using the extracted release version and a lowercase repository image name. |
| Convenience latest tag | Step 14 tags the same build with both the release version and `latest`, pushes both tags, and fixes the build step indentation. |

## Build once, deploy many

Build one Docker image for each release. Test that image in staging, then deploy the same image to production after approval. Changes require a new version; existing release images should not be overwritten.

## Container Registry

We will use GitHub Container Registry (GHCR) to store release images.
GitHub Actions will build and publish these images from our repository.

First planned image: `ghcr.io/alisalman909/mlops-classactivity:1.0.0`

## CD Workflow: Step 10

The workflow belongs in `.github/workflows/cd.yml`. Pushing a tag matching
`v*.*.*`, such as `v1.0.0`, triggers it; ordinary branch pushes do not.
This pattern matches release-style names but does not strictly validate semantic versions.

The test job runs on Ubuntu with Python 3.12 and executes the API tests with pytest.
The workflow declares read access to repository contents and package write
access for the image publishing steps to be added later.

At step 10, the workflow only runs tests. Image building, publishing,
staging deployment, and production approval will be added in later steps.
Step 13 introduces the first release tag to test image publishing; deployment comes later.

## Release Version: Step 11

The `build` job depends on successful completion of the `test` job.
For a pushed tag such as `v1.3.0`, `${GITHUB_REF_NAME#v}` extracts `1.3.0`.
The `Get version` step writes the value to `GITHUB_OUTPUT`; the job exposes
it as its `version` output so later deployment jobs can use the same version.

This version comes from the Git tag, not the `VERSION` file. Keep the file
and release tag consistent when releasing. At step 11, the build job only
extracts and displays the version; Docker image building is added later.

## GHCR Login: Step 12

Add the login step after `Show version` in the build job. It authenticates
to `ghcr.io` using `github.actor` as the username and `secrets.GITHUB_TOKEN`
as the password. GitHub provides this token automatically for the workflow;
no personal password or manually created token is needed for this step.

The existing `packages: write` permission enables package publishing.
Login alone does not build or upload an image; those steps come next.

## Build and Publish: Step 13

After registry login, build the Docker image and push it to GHCR.
`${GITHUB_REPOSITORY,,}` converts the repository path to lowercase for
Docker image naming. The extracted release version supplies the image tag.

After committing and pushing the workflow, push Git tag `v1.0.0` to trigger
tests, image building, and publishing of
`ghcr.io/alisalman909/mlops-classactivity:1.0.0`.
Check GitHub Actions for the run result. Staging and production deployment
are not configured yet. Use a new release version for future changes
rather than moving an existing release tag.

## Convenience Tag: Step 14

The workflow builds one image with two tags: the extracted release version
and `latest`. Both tags are pushed to GHCR. `latest` is a movable label
updated by each successful push of that tag; it is not a separate build
or a guarantee of the highest version number.

Use explicit release versions for production and rollback, keeping release
tags unchanged. Committing this workflow change does not publish images;
it takes effect on the next new release tag. Do not recreate `v1.0.0`
if it has already been pushed.

## GitHub Environments: Step 15

Setup is performed in the repository's Settings > Environments page:

1. Create an environment named `staging` without required reviewers.
2. Create an environment named `production`. The required-reviewer
   approval gate will be configured in step 21.

Environments group deployment secrets, variables, protection rules, and
deployment history. They do not create servers or deploy the application.
Server setup, environment secrets, and workflow deployment jobs are added
in later steps. Creating an environment named `production` alone does not
enable an approval gate.

These settings are stored on GitHub, outside Git history. This README
documents the setup procedure; verify that both environments appear in
GitHub before considering the setup complete.

## Local Deployment Host: Adapted Step 16

This exercise uses Docker Desktop on Windows with a repository-level
self-hosted GitHub Actions runner instead of an Ubuntu server and SSH.
The user confirmed that the runner shows `Listening for Jobs` and that
`docker info` reports both client and server information. This verifies
runner registration and Docker access; an actual deployment job still
needs to succeed to verify the complete path.

Setup procedure:

1. Keep Docker Desktop running with Linux containers.
2. Open repository Settings > Actions > Runners > New self-hosted runner,
   and select Windows and the matching architecture (normally x64).
3. Use GitHub's generated download and configuration commands in a runner
   folder outside this repository, such as `%USERPROFILE%\actions-runner`.
4. Name the runner `mlops-laptop`, add the custom label `mlops-local`,
   accept the default runner group and work folder, and decline service
   installation. Run it interactively under the same Windows account
   that uses Docker Desktop with `.\run.cmd`.
5. Confirm `Listening for Jobs` locally and `Idle` on the GitHub runner
   page. Verify Docker access from that Windows account with `docker info`.

Keep the runner terminal and Docker Desktop running for deployments.
Do not commit the runner installation or registration credentials.
Use the local runner only for trusted release deployments, not pull request
jobs; workflow code on it can access the laptop. CI and image builds stay
on GitHub-hosted Ubuntu runners. Local deployment jobs will use PowerShell.

Later steps will use separate containers and localhost ports for staging
and simulated production on this laptop. This is a classroom simulation,
not separate production infrastructure. GitHub environments still provide
deployment settings and production approval. SSH setup and host/key secrets
from the Ubuntu approach are replaced by local runner execution.

## Local Access: Adapted Steps 17 and 18

The runner is installed at `C:\Users\Ali's HP\actions-runner` and runs
interactively under the Windows user account that has Docker access.
It receives jobs from GitHub and executes Docker locally, so this setup
does not use SSH keys or `STAGING_HOST`, `STAGING_USER`, and
`STAGING_SSH_KEY` secrets. These tutorial steps are replaced, not performed
as Ubuntu/SSH setup. The GitHub `staging` environment remains in use.

## Automatic Staging Deployment: Adapted Step 19

`deploy-staging` waits for the build job and targets runner labels
`self-hosted`, `Windows`, `X64`, and `mlops-local`. Ensure the registered
runner has all four labels in Settings > Actions > Runners.

The job authenticates to GHCR with `GITHUB_TOKEN` and `packages: read`,
using a separate temporary Docker configuration. It downloads the exact
release-tagged image produced by the build job without rebuilding it.
After pulling successfully, it replaces only the `mlops-staging` container.
Concurrent staging deployment jobs are serialized.

Staging is available on the laptop at `http://127.0.0.1:5001/health`.
Port 5001 on Windows maps to port 5000 inside the container and is bound
to localhost. Deployment scripts use Windows PowerShell with `-File` to
avoid the default shell command's quoting issue with `Ali's HP` paths.
This does not fix the separate startup message from `run.cmd`.

Keep Docker Desktop and the runner terminal open. After committing the
workflow, a new release tag triggers tests, publishing, and deployment.
Use an unused version and keep `VERSION` consistent with it. Step 19 has
been configured locally; its workflow run has not yet been verified.
Step 20 adds the automated health check. Production approval and deployment
are still to be added in subsequent steps.

## Staging Smoke Test: Adapted Step 20

After starting the container, the same Windows runner requests
`http://127.0.0.1:5001/health` using PowerShell. The check requires HTTP 200
and a JSON body with `status` equal to `healthy`.

The check makes up to 12 attempts, with a five-second request timeout and
five-second pauses between failed attempts, allowing time for app startup.
If all attempts fail, it throws an error and marks `deploy-staging` failed.
Failure does not automatically stop the container or roll it back.

The future production job must depend on successful staging deployment
so that this check gates promotion. It does not measure model accuracy.
The smoke test is configured; a release workflow run must still verify it.
