# MLOps Continuous Delivery Demo

A tutorial project for learning how to test, package, and release a small Flask API using Docker and GitHub Actions. The demo prediction doubles the input number; it does not use a trained ML model.

## Progress by commit

The summaries below describe the completed steps; commit titles follow the commands used during the tutorial.

| Commit / step | What we did |
| --- | --- |
| Flask API and dependencies | Added `app.py` with `/`, `/health`, and `/predict` endpoints, plus Flask and pytest dependencies in `requirements.txt`. |
| API tests and requirements fix | Added tests for the health endpoint and prediction result. Removed terminal commands from `requirements.txt`, installed dependencies in a virtual environment, and ran the tests. |
| Docker containerization | Added a Dockerfile using Python 3.12, built the local image, ran the container, and checked `/health`. |
| Documentation (this step) | Documented our progress and the build-once, deploy-many principle. |

## Build once, deploy many

Build one Docker image for each release. Test that image in staging, then deploy the same image to production after approval. Changes require a new version; existing release images should not be overwritten.

## Container Registry

We will use GitHub Container Registry (GHCR) to store release images.
GitHub Actions will build and publish these images from our repository.

First planned image: `ghcr.io/alisalman909/mlops-classactivity:1.0.0`
