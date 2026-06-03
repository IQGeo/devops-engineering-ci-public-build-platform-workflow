# Who Calls What

## Scope

This document shows the inbound and outbound workflow dependencies for `devops-engineering-ci-public-build-platform-workflow`.

## Matrix

| Direction | Component | Relationship | Notes |
| --- | --- | --- | --- |
| Called by | Caller workflow or product pipeline | Inbound | Typically called after `devops-engineering-ci-public-extract-version-action` has produced the normalized version and tag outputs |
| Calls | `.github/workflows/cut-platform.yml` | Direct | Produces platform cut artifacts used by downstream builds |
| Calls | `devops-engineering-ci-public-build-multi-arch-workflow/.github/workflows/build-multi-arch.yml` | Direct | Builds injector images for each configured module |
| Calls | `devops-engineering-ci-public-build-platform-specialised-image-workflow/.github/workflows/build-specialised-images.yml` | Direct | Builds the specialised platform image family in waves |
| Calls | `devops-engineering-ci-public-build-platform-specialised-image-workflow/.github/workflows/build-devdb-qa-images.yml` | Optional | Builds the DevDB QA image chain when `build_qa_images=true` |
| Calls | `devops-engineering-ci-redeploy-eks-pod/.github/workflows/redeploy-eks-pod.yml` | Optional | Invoked after the DevDB QA image chain when redeploy inputs are provided |
| Adjacent | `.github/workflows/build-prereqs-minimal.yml` | Same repo support | This repo contains the pre-req image workflow, but `build-platform.yml` does not directly call it in the current implementation |

## Notes

- The platform workflow is the main orchestration layer for the IQGeo Platform image family.
- Specialised platform images and DevDB QA images are delegated to the supporting specialised-image workflow repo.