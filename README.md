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

The test job runs on Ubuntu with Python 3.12 and executes `python -m pytest`.
The workflow declares read access to repository contents and package write
access for the image publishing steps to be added later.

At this stage, the workflow only runs tests. Image building, publishing,
staging deployment, and production approval will be added in later steps.
The first release tag will be pushed after the intended release workflow is ready.
