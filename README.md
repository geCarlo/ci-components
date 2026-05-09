# SBOM Pipeline Component

A reusable GitLab CI component that scans a container image for dependencies,
converts the results to CSV, and processes them using your own internal tooling.

## Jobs

| Job | Stage | Image | What it does |
|-----|-------|-------|--------------|
| `<name>-sbom-scan` | scan | `aquasec/trivy` | Scans the target image and produces a CycloneDX SBOM |
| `<name>-sbom-to-csv` | convert | `python:3.12-slim` | Converts the CycloneDX SBOM to a CSV of components |
| `<name>-process-csv` | process | your `sbom-processor` image | Processes the CSV with your own tooling |

## Usage

```yaml
include:
  - component: gitlab.com/<namespace>/<project>/pipeline@~latest
    inputs:
      name: my-app
      image: registry.gitlab.com/my-namespace/my-app:latest

stages:
  - sbom
```

### With all inputs

```yaml
include:
  - component: gitlab.com/<namespace>/<project>/pipeline@v1.0.0
    inputs:
      name: my-app
      image: registry.gitlab.com/my-namespace/my-app:$CI_COMMIT_SHORT_SHA
      trivy_version: "0.58.0"
      severity: "HIGH,CRITICAL"
      processor_version: "1.0.0"

stages:
  - sbom
```

### Multiple images in the same pipeline

Include the component once per image, using a unique `name` for each:

```yaml
include:
  - component: gitlab.com/<namespace>/<project>/pipeline@v1.0.0
    inputs:
      name: api
      image: registry.gitlab.com/my-namespace/my-app/api:$CI_COMMIT_SHORT_SHA

  - component: gitlab.com/<namespace>/<project>/pipeline@v1.0.0
    inputs:
      name: worker
      image: registry.gitlab.com/my-namespace/my-app/worker:$CI_COMMIT_SHORT_SHA

stages:
  - sbom
```

### Full Example

```yaml
include:
  - component: gitlab.com/<namespace>/ci-compo
nents/pipeline@v1.0.0
    inputs:
      name: my-app
      image:
$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    rules:
      - if: $CI_COMMIT_TAG

stages:
  - build
  - test
  - release
  - sbom

build:
  stage: build
  script:
    - ...

test:
  stage: test
  script:
    - ...

image-release:
  stage: release
  script:
    - docker build --tag
$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push
$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  rules:
    - if: $CI_COMMIT_TAG
```

## Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | *(required)* | Unique identifier for this inclusion — prefixes all job names |
| `image` | string | *(required)* | Container image to scan |
| `trivy_version` | string | `latest` | Trivy image version |
| `severity` | string | `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL` | Severity levels to include in the SBOM |
| `processor_version` | string | `latest` | Version tag of your `sbom-processor` image |

## Publishing a new version

Requires [`git-cliff`](https://github.com/orhun/git-cliff/releases) installed locally.

```bash
# 1. Generate the changelog (--bump auto-picks the next semver version)
git cliff --bump -o CHANGELOG.md

# 2. Commit, tag, and push
VERSION=$(git cliff --bumped-version)
git commit -am "chore: release $VERSION"
git tag $VERSION
git push origin main --follow-tags
```

CI picks up the tag and handles the rest: builds the image, extracts the
release notes from `CHANGELOG.md`, and publishes the component to the GitLab
CI Catalog.

> **One-time setup:** mark the project as a catalog resource under
> **Settings → General → Visibility → CI/CD Catalog resource**.
