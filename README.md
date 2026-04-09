# GitHub Actions Templates (.NET)

Reusable GitHub Actions workflows for .NET projects.

This repository provides `workflow_call` templates that can be referenced from other repositories to standardize CI/CD steps such as build, unit tests, integration tests, mutation tests, SonarQube Cloud analysis, packaging, and publishing.

## Table of contents

- [Available templates](#available-templates)
  - [`dotnet-build.yml`](#dotnet-buildyml)
  - [`dotnet-unit-test.yml`](#dotnet-unit-testyml)
  - [`dotnet-unit-test-with-sonar-scanner.yml`](#dotnet-unit-test-with-sonar-scanneryml)
  - [`dotnet-integration-test.yml`](#dotnet-integration-testyml)
  - [`dotnet-mutation-test.yml`](#dotnet-mutation-testyml)
  - [`dotnet-pack.yml`](#dotnet-packyml)
  - [`dotnet-publish.yml`](#dotnet-publishyml)
  - [`dotnet-validate-bff.yml`](#dotnet-validate-bffyml)
  - [`dotnet-validate-contracts.yml`](#dotnet-validate-contractsyml)
  - [`dotnet-validate-full.yml`](#dotnet-validate-fullyml)
- [How to use from another repository](#how-to-use-from-another-repository)
  - [Example: Full validation](#example-full-validation)
  - [Example: BFF validation](#example-bff-validation)
  - [Example: Publish package](#example-publish-package)
- [Required secrets in caller repository](#required-secrets-in-caller-repository)
- [Notes](#notes)

---

## Available templates

All templates live in `.github/workflows/`.

### `dotnet-build.yml`

Builds a solution/project.

**Inputs**

- `dotnet_version` (string, default: `10.x`)
- `build_mode` (string, default: `Release`)
- `build_path` (string, default: `.`)

---

### `dotnet-unit-test.yml`

Runs unit tests.

**Inputs**

- `test_project_path` (string, required)
- `dotnet_version` (string, default: `10.x`)
- `no_build` (boolean, default: `true`)
- `build_configuration` (string, default: `Release`)
- `verbosity` (string, default: `n`)
- `name` (string, default: `Unit Tests`)

---

### `dotnet-unit-test-with-sonar-scanner.yml`

Runs unit tests with SonarQube Cloud analysis (Windows runner + JDK 17 + `dotnet-sonarscanner`).

**Inputs**

- `organization` (string, default: repository owner)
- `project` (string, default: repository name)
- `dotnet_version` (string, default: `10.x`)
- `unit_test_verbosity` (string, default: `n`)
- `unit_test_project_path` (string, required)
- `sonar_inclusions` (string, default: `**src/Application**,**src/Domain**`)
- `sonar_host_url` (string, default: `https://sonarcloud.io`)
- `no_build` (boolean, default: `true`)
- `build_configuration` (string, default: `Release`)

**Secrets**

- `sonar_token` (required)

---

### `dotnet-integration-test.yml`

Starts Docker Compose and runs integration tests.

**Inputs**

- `docker_compose_file_path` (string, required)
- `test_project_path` (string, required)
- `dotnet_version` (string, default: `10.x`)
- `build_configuration` (string, default: `Release`)
- `verbosity` (string, default: `n`)
- `no_build` (boolean, default: `true`)

---

### `dotnet-mutation-test.yml`

Runs mutation tests using Stryker.

**Inputs**

- `test_project_path` (string, required)
- `stryker_config_path` (string, required)
- `dotnet_version` (string, default: `8.x`)
- `log_level` (string, default: `info`)
- `name` (string, default: `Mutation Tests`)
- `publish_dashboard_report` (boolean, default: `false`)
- `dashboard_project` (string, optional, default: `''`)
- `dashboard_version` (string, optional, default: `''`)
- `dashboard_module` (string, optional, default: `''`)
- `dashboard_base_url` (string, default: `https://dashboard.stryker-mutator.io`)

**Secrets**

- `dashboard_api_key` (optional; required when `publish_dashboard_report: true`)

---

### `dotnet-pack.yml`

Builds, packs, and pushes a NuGet package.

**Inputs**

- `package_version` (string, required)
- `dotnet_version` (string, default: `10.x`)

**Secrets**

- `nuget_api_key` (required)

---

### `dotnet-publish.yml`

Composes build + pack/publish by reusing `dotnet-build.yml` and `dotnet-pack.yml`.

**Inputs**

- `build_path` (string, default: `.`)
- `dotnet_version` (string, default: `10.x`)
- `package_version` (string, required)

**Secrets**

- `nuget_api_key` (required)

---

### `dotnet-validate-bff.yml`

Validation pipeline for BFF-style services:

- build
- optional integration tests

**Inputs**

- `build_path` (string, default: `.`)
- `dotnet_version` (string, default: `10.x`)
- `test_no_build` (boolean, default: `true`)
- `integration_test_project_path` (string, optional)
- `docker_compose_file_path` (string, optional)
- `test_verbosity` (string, default: `n`)

---

### `dotnet-validate-contracts.yml`

Validation pipeline for contracts-focused repositories:

- build
- unit tests
- optional mutation tests

**Inputs**

- `build_path` (string, default: `.`)
- `dotnet_version` (string, default: `10.x`)
- `stryker_enable` (boolean, default: `true`)
- `stryker_config_path` (string, optional)
- `stryker_log_level` (string, default: `info`)
- `stryker_publish_dashboard_report` (boolean, default: `false`)
- `stryker_dashboard_project` (string, optional, default: `''`)
- `stryker_dashboard_version` (string, optional, default: `''`)
- `stryker_dashboard_module` (string, optional, default: `''`)
- `stryker_dashboard_base_url` (string, default: `https://dashboard.stryker-mutator.io`)
- `test_no_build` (boolean, default: `true`)
- `test_project_path` (string, optional)
- `test_verbosity` (string, default: `n`)

**Secrets**

- `stryker_dashboard_api_key` (optional; required only when Stryker dashboard publishing is enabled)

---

### `dotnet-validate-full.yml`

Full validation pipeline:

- build
- optional unit tests with SonarQube Cloud
- optional domain/application mutation tests
- optional integration tests

**Inputs**

- `build_path` (string, default: `.`)
- `dotnet_version` (string, default: `10.x`)
- `stryker_enable` (boolean, default: `true`)
- `test_no_build` (boolean, default: `true`)
- `unit_test_project_path` (string, optional)
- `stryker_domain_config_path` (string, optional)
- `stryker_application_config_path` (string, optional)
- `stryker_log_level` (string, default: `info`)
- `stryker_publish_dashboard_report` (boolean, default: `false`)
- `stryker_dashboard_project` (string, optional, default: `''`)
- `stryker_dashboard_version` (string, optional, default: `''`)
- `stryker_dashboard_module_domain` (string, optional, default: `''`)
- `stryker_dashboard_module_application` (string, optional, default: `''`)
- `stryker_dashboard_base_url` (string, default: `https://dashboard.stryker-mutator.io`)
- `integration_test_project_path` (string, optional)
- `docker_compose_file_path` (string, optional)
- `unit_test_verbosity` (string, default: `n`)
- `sonar_organization` (string, default: repository owner)
- `sonar_validation` (boolean, default: `true`)
- `sonar_project` (string, default: repository name)
- `sonar_host_url` (string, default: `https://sonarcloud.io`)

**Secrets**

- `sonar_token` (optional unless Sonar validation is enabled and unit tests are executed)
- `stryker_dashboard_api_key` (optional; required only when Stryker dashboard publishing is enabled)

---

## How to use from another repository

In your consuming repository, create a workflow that calls one of these templates.

> Replace `<owner>`, `<repo>`, and `<ref>` (branch/tag/SHA) with your values.

### Example: Full validation

```yaml
name: Validate

on:
  pull_request:
  push:
    branches: [ main ]

jobs:
  validate:
    uses: <owner>/<repo>/.github/workflows/dotnet-validate-full.yml@<ref>
    with:
      dotnet_version: '10.x'
      build_path: '.'
      unit_test_project_path: tests/MyProject.UnitTests/MyProject.UnitTests.csproj
      stryker_domain_config_path: tests/MyProject.UnitTests/stryker-domain.json
      stryker_application_config_path: tests/MyProject.UnitTests/stryker-application.json
      integration_test_project_path: tests/MyProject.IntegrationTests/MyProject.IntegrationTests.csproj
      docker_compose_file_path: docker-compose.yml
      sonar_organization: my-sonar-org
      sonar_project: my-sonar-project
      sonar_host_url: https://sonarcloud.io
      sonar_validation: true
      stryker_enable: true
      stryker_log_level: info
      stryker_publish_dashboard_report: true
      stryker_dashboard_project: github.com/my-org/my-repo
      stryker_dashboard_version: ${{ github.ref_name }}
      stryker_dashboard_module_domain: domain
      stryker_dashboard_module_application: application
      stryker_dashboard_base_url: https://dashboard.stryker-mutator.io
      unit_test_verbosity: n
      test_no_build: true
    secrets:
      sonar_token: ${{ secrets.SONAR_TOKEN }}
      stryker_dashboard_api_key: ${{ secrets.STRYKER_DASHBOARD_API_KEY }}
```

### Example: BFF validation

```yaml
name: Validate BFF

on:
  pull_request:

jobs:
  validate:
    uses: <owner>/<repo>/.github/workflows/dotnet-validate-bff.yml@<ref>
    with:
      dotnet_version: '10.x'
      build_path: '.'
      integration_test_project_path: tests/MyProject.IntegrationTests/MyProject.IntegrationTests.csproj
      docker_compose_file_path: docker-compose.yml
      test_no_build: true
      test_verbosity: n
```

### Example: Publish package

```yaml
name: Publish

on:
  workflow_dispatch:

jobs:
  publish:
    uses: <owner>/<repo>/.github/workflows/dotnet-publish.yml@<ref>
    with:
      dotnet_version: '10.x'
      build_path: '.'
      package_version: 1.2.3
    secrets:
      nuget_api_key: ${{ secrets.NUGET_API_KEY }}
```

---

## Required secrets in caller repository

Depending on the template used, configure and map:

- `SONAR_TOKEN` → `sonar_token`
- `NUGET_API_KEY` → `nuget_api_key`
- `STRYKER_DASHBOARD_API_KEY` → `dashboard_api_key` (for `dotnet-mutation-test.yml`)
- `STRYKER_DASHBOARD_API_KEY` → `stryker_dashboard_api_key` (for validate templates)

(Secret names in your repository can differ, as long as they are mapped in the `secrets:` block of the calling workflow.)

---

## Notes

- `dotnet-unit-test-with-sonar-scanner.yml` runs on `windows-latest`; most other templates run on `ubuntu-latest`.
- Validation templates default to `.NET 10.x`; `dotnet-mutation-test.yml` currently defaults to `.NET 10.x`.
- Pin reusable workflow references to a tag or commit SHA for safer, reproducible pipelines.

Happy automating 🚀
