# GitHub Actions Templates (.NET)

Reusable GitHub Actions workflows for .NET projects.

This repository provides a set of `workflow_call` templates you can reference from other repositories to standardize CI/CD steps like build, tests, mutation tests, Sonar analysis, packaging, and publishing.

---

## Available Templates

All templates live in `.github/workflows/`.

### `dotnet-build.yml`

Builds the solution/project.

#### Inputs (`dotnet-build.yml`)

- `dotnet_version` (string, required, default: `8.x`)
- `build_mode` (string, optional, default: `Release`)
- `build_path` (string, optional, default: `.`)

---

### `dotnet-unit-test.yml`

Runs unit tests.

#### Inputs (`dotnet-unit-test.yml`)

- `test_project_path` (string, required)
- `dotnet_version` (string, required, default: `8.x`)
- `no_build` (boolean, optional, default: `false`)
- `build_configuration` (string, optional, default: `Release`)
- `name` (string, optional, default: `Unit Tests`)

---

### `dotnet-unit-test-with-sonar-scanner.yml`

Runs unit tests with SonarQube Cloud analysis (Windows runner + JDK 17 + dotnet-sonarscanner).

#### Inputs (`dotnet-unit-test-with-sonar-scanner.yml`)

- `organization` (string, optional, default: repository owner)
- `project` (string, optional, default: repository name)
- `dotnet_version` (string, optional, default: `8.x`)
- `unit_test_verbosity` (string, optional, default: `n`)
- `unit_test_project_path` (string, required)
- `sonar_inclusions` (string, optional, default: `**src/Application**,**src/Domain**`)
- `sonar_host_url` (string, optional, default: `https://sonarcloud.io`)
- `no_build` (boolean, optional, default: `false`)
- `build_configuration` (string, optional, default: `Release`)

#### Secrets (`dotnet-unit-test-with-sonar-scanner.yml`)

- `sonar_token` (required)

---

### `dotnet-integration-test.yml`

Starts Docker Compose and runs integration tests.

#### Inputs (`dotnet-integration-test.yml`)

- `docker_compose_file_path` (string, required)
- `test_project_path` (string, required)
- `dotnet_version` (string, required, default: `8.x`)
- `build_configuration` (string, optional, default: `Release`)

---

### `dotnet-mutation-test.yml`

Runs mutation tests using Stryker.

#### Inputs (`dotnet-mutation-test.yml`)

- `test_project_path` (string, required)
- `stryker_config_path` (string, required)
- `dotnet_version` (string, required, default: `8.x`)
- `log_level` (string, optional, default: `info`)
- `name` (string, optional, default: `Mutation Tests`)

---

### `dotnet-pack.yml`

Builds, packs, and pushes NuGet package.

#### Inputs (`dotnet-pack.yml`)

- `package_version` (string, required)
- `dotnet_version` (string, required, default: `8.x`)

#### Secrets (`dotnet-pack.yml`)

- `nuget_api_key` (required)

---

### `dotnet-validate.yml`

Composite validation pipeline:

- build
- unit tests + Sonar
- optional mutation tests (domain/application) — enabled via `stryker_enable`; skipped if `unit_test_project_path` is empty
- optional integration tests — skipped if `integration_test_project_path` is empty

#### Inputs (`dotnet-validate.yml`)

- `build_path` (string, optional, default: `.`)
- `stryker_enable` (boolean, optional, default: `true`)
- `test_no_build` (boolean, optional, default: `true`)
- `unit_test_project_path` (string, optional)
- `domain_stryker_config_path` (string, optional)
- `application_stryker_config_path` (string, optional)
- `stryker_log_level` (string, optional, default: `info`)
- `integration_test_project_path` (string, optional)
- `docker_compose_file_path` (string, optional)
- `dotnet_version` (string, required, default: `8.x`)
- `unit_test_verbosity` (string, optional, default: `n`)
- `sonar_organization` (string, optional, default: repository owner)
- `sonar_project` (string, optional, default: repository name)
- `sonar_host_url` (string, optional, default: `https://sonarcloud.io`)

#### Secrets (`dotnet-validate.yml`)

- `sonar_token` (required)

---

### `dotnet-publish.yml`

Builds and publishes NuGet package.

This workflow is intentionally minimal and currently includes:

- build
- NuGet pack + publish

#### Inputs (`dotnet-publish.yml`)

- `build_path` (string, optional, default: `.`)
- `dotnet_version` (string, required, default: `8.x`)
- `package_version` (string, required)

#### Secrets (`dotnet-publish.yml`)

- `nuget_api_key` (required)

---

## How to Use from Another Repository

In your consuming repository, create a workflow that calls one of these templates.

> Replace `<owner>`, `<repo>`, and `<ref>` (branch/tag/SHA) with your values.

### Example: Validate pipeline

```yaml
name: Validate

on:
  pull_request:
  push:
    branches: [ main ]

jobs:
  validate:
    uses: <owner>/<repo>/.github/workflows/dotnet-validate.yml@<ref>
    with:
      dotnet_version: '8.x'
      build_path: '.'
      unit_test_project_path: tests/MyProject.UnitTests/MyProject.UnitTests.csproj
      domain_stryker_config_path: tests/MyProject.UnitTests/stryker-domain.json
      application_stryker_config_path: tests/MyProject.UnitTests/stryker-application.json
      integration_test_project_path: tests/MyProject.IntegrationTests/MyProject.IntegrationTests.csproj
      docker_compose_file_path: docker-compose.yml
      sonar_organization: my-sonar-org
      sonar_project: my-sonar-project
      sonar_host_url: https://sonarcloud.io
      stryker_enable: true
      stryker_log_level: info
      unit_test_verbosity: n
      test_no_build: true
    secrets:
      sonar_token: ${{ secrets.SONAR_TOKEN }}
```

### Example: Publish pipeline

```yaml
name: Publish

on:
  workflow_dispatch:

jobs:
  publish:
    uses: <owner>/<repo>/.github/workflows/dotnet-publish.yml@<ref>
    with:
      dotnet_version: '8.x'
      build_path: '.'
      package_version: 1.2.3
    secrets:
      nuget_api_key: ${{ secrets.NUGET_API_KEY }}
```

---

## Required Secrets in Caller Repository

Depending on the template used, configure:

- `SONAR_TOKEN`
- `NUGET_API_KEY`

(Names in your repository secrets can differ, but must be mapped in the `secrets:` block of the calling workflow.)

---

## Notes

- `dotnet-unit-test-with-sonar-scanner.yml` runs on `windows-latest`; others mostly run on `ubuntu-latest`.
- Pin reusable workflow references to a tag or commit SHA for safer, reproducible pipelines.
- Use `dotnet-validate.yml` for quality gates. Use `dotnet-publish.yml` when you only need build + package publish.

Happy automating 🚀
