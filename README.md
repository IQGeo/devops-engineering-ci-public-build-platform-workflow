# devops-engineering-ci-public-build-platform-workflow

Reusable GitHub Actions workflows for building IQGeo Platform, its injector images, and its related QA image chain.

## Overview

This repository contains the top-level reusable workflow used for platform builds. It orchestrates the wider platform image family, but delegates repeated work to the supporting workflows and actions in the other CI repositories.

The main flow is:

1. Cut platform build artifacts from source.
2. Build injector images for the configured module set.
3. Build the specialised platform image family in dependency order.
4. Optionally build DevDB QA images.
5. Optionally redeploy PMG pods after QA images are available.

For inbound and outbound dependency relationships, see [docs/WHO-CALLS-WHAT.md](docs/WHO-CALLS-WHAT.md).

## Workflows

### `.github/workflows/build-platform.yml`

This is the primary reusable entry point.

Key responsibilities:

- Accepts the target version, tags, build id, module list, and release intent.
- Calls `.github/workflows/cut-platform.yml` to package binaries.
- Calls `devops-engineering-ci-public-build-multi-arch-workflow` to build injector images.
- Calls `devops-engineering-ci-public-build-platform-specialised-image-workflow` to build specialised platform images.
- Calls the DevDB QA image workflow when `build_qa_images` is enabled.
- Can call `devops-engineering-ci-redeploy-eks-pod` after the QA image chain succeeds.

Typical flow:

```text
caller workflow
	-> build-platform.yml
		 -> cut-platform.yml
		 -> build-multi-arch.yml for injector images
		 -> build-specialised-images.yml for platform base/build/appserver/tools/devenv images
		 -> optional build-devdb-qa-images.yml
		 -> optional redeploy-eks-pod.yml
```

### `.github/workflows/cut-platform.yml`

Packages the source artifacts required by downstream platform image builds.

Key responsibilities:

- Checks out `IQGeo/myworld-product-core` and `IQGeo/myworld-product-native-apps`.
- Installs the Python dependencies required by the platform cut script.
- Runs `cut_all` to generate the binaries archive set.
- Uploads those artifacts to Azure File Share and as a GitHub Actions artifact.

### `.github/workflows/build-prereqs-minimal.yml`

Builds a minimal multi-architecture pre-requisites image directly with Docker Buildx.

Key responsibilities:

- Builds amd64 and arm64 images for the requested Dockerfile.
- Creates the multi-arch manifest in ACR.
- Provides the lower-level pre-req image consumed by later platform builds.

## Important inputs

The main platform workflow depends on these inputs:

- `version`: platform version being built.
- `modules`: comma-separated injector module list.
- `shortened_version`: version shorthand used for language pack lookup.
- `tags`: comma-separated tags to apply to final multi-arch images.
- `build_id`: run-specific identifier used for architecture-specific tags.
- `preqs_tag`: pre-req image tag used by specialised builds.
- `pip_flags`: optional pip flags passed into the platform image build.
- `is_release`: controls whether release repositories are retagged.
- `build_qa_images`: toggles the DevDB QA image chain.
- `namespace` and `pod_name`: optional post-build redeploy settings.

## How this repo fits the wider build stack

- Upstream: caller workflows usually run `devops-engineering-ci-public-extract-version-action` first, then call this workflow.
- Downstream: this workflow depends on the shared multi-arch workflow, the specialised image workflow repo, and the EKS redeploy workflow.
- Scope: this repo owns platform build orchestration, not the low-level Docker build logic.
