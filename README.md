# devops-engineering-ci-public-build-platform-workflow

This repository contains reusable GitHub Actions workflows for building IQGeo Platform images and related module components. The workflows are designed to be called from other repositories or workflows, providing a standardized, modular, and automated build process for platform releases and development builds.

## Workflows Overview

### 1. `.github/workflows/build-platform.yml` — **Reusable Build Workflow Template**

This is the main entrypoint workflow for building platform images and modules. It is designed to be called using `workflow_call` from other workflows or repositories, and orchestrates the entire build process.

**Key Features:**
- Accepts version, module list, tags, and other build parameters as inputs.
- Sets up build variables and processes module names for downstream jobs.
- Calls the `cut-platform.yml` workflow to prepare source code artifacts for the build.
- Triggers downstream workflows to build multi-architecture images for modules and platform components (base, build, appserver, tools, devenv, devdb-qa, etc.).
- Optionally redeploys a Kubernetes pod after images are built.

**Downstream Workflows Called:**
- `cut-platform.yml` (in this repo): Prepares and uploads source code artifacts for the build.
- `IQGeo/devops-engineering-ci-public-build-multi-arch-workflow/.github/workflows/build-multi-arch.yml@main`: Builds multi-arch Docker images for each module.
- `IQGeo/devops-engineering-ci-public-build-platform-specialised-image-workflow/.github/workflows/build-specialised-images.yml@main`: Builds specialized platform images (base, build, appserver, tools, devenv, etc.).
- `IQGeo/devops-engineering-ci-public-build-platform-specialised-image-workflow/.github/workflows/build-devdb-qa-images.yml@main`: Builds QA images (platform with devdb), if enabled.
- `IQGeo/devops-engineering-ci-redeploy-eks-pod/.github/workflows/redeploy-eks-pod.yml@main`: Optionally redeploys a Kubernetes pod after QA images are built.

**Inputs:**
- `version`: Version to build (required)
- `modules`: Comma-separated list of modules to build (default provided)
- `shortened_version`: Short version string for language packs (required)
- `tags`: Comma-separated list of tags to apply (required)
- `build_id`: Unique build identifier (required)
- `engineering_prefix`, `releases_prefix`: Prefixes for image placement (defaults provided)
- `is_release`: Whether this is a release or pre-release (required)
- `namespace`, `pod_name`: For optional Kubernetes redeploy
- `build_qa_images`: Whether to build QA images (default: true)

**Secrets:**
- `GH_TOKEN`, `HARBOR_USERNAME`, `HARBOR_CLI_SECRET`, `REGISTRY_USERNAME`, `REGISTRY_PASSWORD`

### 2. `.github/workflows/cut-platform.yml` — **Cut Module Workflow**

This workflow is responsible for preparing a cut (snapshot) of the source code for a specified version. It checks out the relevant repositories, runs a Python script to generate binaries/artifacts, and uploads them to Azure File Share for use in downstream builds.

**Key Features:**
- Checks out the core and native apps repositories at the specified version.
- Installs required Python dependencies and runs the `cut_all` script to generate artifacts.
- Uploads the generated binaries to Azure File Share for later use in image builds.
- Uploads the binaries as a GitHub Actions artifact for traceability.

**Inputs:**
- `version`: Version to build (required)

**Secrets:**
- `GH_TOKEN`, `AZURE_STORAGE_ACCOUNT_KEY`, `AZURE_CREDENTIALS`

**Typical Flow:**
1. The `build-platform.yml` workflow is triggered (via `workflow_call`).
2. It calls `cut-platform.yml` to prepare and upload source code artifacts.
3. Once artifacts are ready, it triggers downstream workflows to build and push Docker images for each module and platform component, using the prepared artifacts.
4. Optionally, QA images are built and a Kubernetes pod is redeployed if configured.

---

For more details, see the comments and documentation within each workflow file in `.github/workflows/`.
