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
