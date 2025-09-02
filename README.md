# PipelineOps — Reusable CI/CD Components

### *Exploring how complex engineering challenges can be solved at scale.*

---

## What this project is

**PipelineOps** is a library of **GitLab CI/CD components** designed to standardize container builds across projects.

The current component, **`buildx-build`**, is a **fully parameterized job template** for building and pushing Docker images using **Docker Buildx**. It allows any project to define a build job simply by including the template and passing in inputs (like image name, tag, and Dockerfile path).

This turns repetitive build jobs into a **single, reusable standard** — making pipelines cleaner, faster, and easier to maintain.

---

## Inputs this component accepts

Your template exposes a set of **inputs** so projects can adapt it to their needs:

* **`job_name`** → sets the GitLab job name (default: `buildx-build`).
* **`job_stage`** → which pipeline stage to run in (default: `.pre`).
* **`docker_path`** → relative path to the directory containing the Dockerfile.
* **`docker_file`** → filename of the Dockerfile (default: `Dockerfile`).
* **`buildx_context`** → Buildx context (e.g., `default`, `remote`, custom).
* **`buildx_instance`** → name of the Buildx builder instance.
* **`buildx_driver`** → driver to use (`docker`, `docker-container`, `kubernetes`).
* **`build_target`** → optional multistage target.
* **`image_name`** → full image name (including registry).
* **`image_tag`** → image tag (default: `latest`).
* **`image_context`** → build context path (default: `.`).

---

## What the job does

1. **Authenticates to registry**
   Logs into GitLab’s container registry using `$CI_REGISTRY_USER` and `$CI_REGISTRY_PASSWORD`.

2. **Configures Docker context**
   Uses or creates a Docker Buildx context (`docker context use` / `docker context create`).

3. **Creates a Buildx builder instance**
   Runs `docker buildx create` with the specified driver, sets it active (`--use`), and boots it with `--bootstrap`.

4. **Builds & pushes the image**
   Constructs a `docker buildx build` command with the inputs:

   * `--tag $IMAGE_NAME:$IMAGE_TAG`
   * `--push` → pushes directly to the registry.
   * `-f $DOCKER_PATH/$DOCKER_FILE`
   * Optional `--target` if multistage is specified.
   * Uses `$IMAGE_CONTEXT` as the final build context.

This produces a **pushed Docker image** that is:

* **Immutable** (commit-SHA tags recommended).
* **Reproducible** (consistent inputs).
* **Portable** (works across languages and stacks).

---

## Example usage in a project

In a consuming project’s `.gitlab-ci.yml`:

```yaml
include:
  - project: 'your-group/pipelineops'
    file: 'template.yml'

build_app_image:
  extends: .buildx-build
  variables:
    job_stage: build
    docker_path: src/app
    docker_file: Dockerfile
    image_name: $CI_REGISTRY_IMAGE/app
    image_tag: $CI_COMMIT_SHORT_SHA
    image_context: src/app
```

This defines a **fully working build job** without duplicating boilerplate logic.

---

## Why this matters

* **Consistency** → every project builds images the same way.
* **Reproducibility** → commit-SHA tags + Buildx caching make builds deterministic.
* **Maintainability** → fix or improve the component once, and every project benefits.
* **Enterprise alignment** → parameterized, reusable components are how large orgs scale CI/CD across many teams.

---

## Maintainer

👤 **Sandesh Nataraj**
📧 [sandeshb.nataraj@gmail.com](mailto:sandeshb.nataraj@gmail.com)
