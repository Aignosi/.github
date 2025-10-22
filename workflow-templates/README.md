# DataOps Module Workflow Templates - Usage Guide

## Overview
This repository contains reusable workflow templates for DataOps modules:

1. **Quality Gate Workflow** (`dataops-module-quality-gate.yml`): Comprehensive quality checks including code formatting, linting, type checking, security analysis, testing, and SonarQube analysis.
2. **Release Workflow** (`dataops-module-release.yml`): Automated release creation with encryption and GitHub release management.

---

## Quality Gate Workflow

### How to Use

#### 1. Copy the Template
Copy the `dataops-module-quality-gate.yml` file to your project's `.github/workflows/` directory.

#### 2. Create a Calling Workflow
Create a new workflow file in your project that calls the reusable template:

```yaml
name: Quality Gate

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    types: [ opened, synchronize, reopened ]

jobs:
  quality-gate:
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-quality-gate.yml@main
    with:
      project_name: 'your-module-name'  # Required: Your project/module name
```

#### 3. Required Parameters

| Parameter | Required | Type | Default | Description |
|-----------|----------|------|---------|-------------|
| `project_name` | ✅ Yes | string | - | Name of the project/module to analyze (e.g., 'ingestor', 'processor') |

#### 4. Optional Parameters

| Parameter | Required | Type | Default | Description |
|-----------|----------|------|---------|-------------|
| `python_version` | ❌ No | string | '3.11' | Python version to use for the analysis |
| `app_owner` | ❌ No | string | 'Aignosi' | GitHub App owner for authentication |
| `repositories` | ❌ No | string | 'sientia-dataops-library,sientia-mlops-library' | Comma-separated list of repositories for GitHub App access |

#### 5. Examples

**Basic Usage:**
```yaml
jobs:
  quality-gate:
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-quality-gate.yml@main
    with:
      project_name: 'ingestor'
```

**Advanced Usage:**
```yaml
jobs:
  quality-gate:
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-quality-gate.yml@main
    with:
      project_name: 'processor'
      python_version: '3.12'
      app_owner: 'YourOrganization'
      repositories: 'your-repo1,your-repo2'
```

**Dynamic Project Name:**
```yaml
jobs:
  quality-gate:
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-quality-gate.yml@main
    with:
      project_name: ${{ github.event.repository.name }}
```

---

## Release Workflow

### How to Use

#### 1. Copy the Template
Copy the `dataops-module-release.yml` file to your project's `.github/workflows/` directory.

#### 2. Create a Calling Workflow
Create a new workflow file in your project that calls the reusable template:

```yaml
name: Create Release on Merge to Main

on:
  pull_request:
    types: [closed]
    branches:
      - main

jobs:
  create-release:
    if: github.event.pull_request.merged == true
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-release.yml@main
    with:
      project_name: 'your-module-name'  # Required: Your project/module name
```

#### 3. Required Parameters

| Parameter | Required | Type | Default | Description |
|-----------|----------|------|---------|-------------|
| `project_name` | ✅ Yes | string | - | Name of the project/module to release (e.g., 'ingestor', 'processor') |

#### 4. Optional Parameters

| Parameter | Required | Type | Default | Description |
|-----------|----------|------|---------|-------------|
| `python_version` | ❌ No | string | '3.11' | Python version to use for the release process |
| `release_prefix` | ❌ No | string | 'sientia' | Prefix for the release package name |
| `encrypt_script` | ❌ No | string | 'encrypt.py' | Name of the encryption script to use |
| `ignore_file` | ❌ No | string | '.gitignore' | File containing patterns to ignore during encryption |
| `chunk_size` | ❌ No | string | '100000' | Chunk size for encryption process |

#### 5. Examples

**Basic Usage:**
```yaml
jobs:
  create-release:
    if: github.event.pull_request.merged == true
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-release.yml@main
    with:
      project_name: 'ingestor'
```

**Advanced Usage:**
```yaml
jobs:
  create-release:
    if: github.event.pull_request.merged == true
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-release.yml@main
    with:
      project_name: 'processor'
      python_version: '3.12'
      release_prefix: 'mycompany'
      encrypt_script: 'custom_encrypt.py'
      ignore_file: '.customignore'
      chunk_size: '50000'
```

**Dynamic Project Name:**
```yaml
jobs:
  create-release:
    if: github.event.pull_request.merged == true
    uses: Aignosi/sientia/.github/workflow-templates/dataops-module-release.yml@main
    with:
      project_name: ${{ github.event.repository.name }}
```

---

## Required Secrets
Make sure these secrets are configured in your repository:
- `APP_ID`: GitHub App ID
- `APP_PRIVATE_KEY`: GitHub App private key
- `SONAR_TOKEN`: SonarQube authentication token
- `SONAR_HOST_URL`: SonarQube server URL

## Required Files
Your project should have:
- `requirements.txt`: Runtime dependencies
- `requirements-dev.txt`: Development dependencies (for quality gate)
- `tests/`: Directory containing your test files (for quality gate)
- `{project_name}/`: Directory containing your source code (where `{project_name}` is the value you pass to the `project_name` parameter)
- `encrypt.py`: Encryption script (for release workflow)
- `.gitignore`: File with patterns to ignore during encryption (for release workflow)

## What Each Workflow Does

### Quality Gate Workflow
1. **Code Formatting**: Checks code formatting using Ruff
2. **Code Linting**: Runs linting checks using Ruff
3. **Type Checking**: Performs type checking using mypy
4. **Security Analysis**: Runs security analysis using Bandit
5. **Testing**: Executes tests with pytest and generates coverage reports
6. **SonarQube Analysis**: Performs static code analysis using SonarQube
7. **Version Management**: Automatically determines next version based on branch naming conventions

### Release Workflow
1. **Version Calculation**: Automatically determines next version based on branch naming conventions
2. **Code Encryption**: Encrypts the project code using the specified encryption script
3. **Package Creation**: Creates a zip file with the encrypted code
4. **GitHub Release**: Creates a GitHub release with the encrypted package as an asset
5. **Branch-based Versioning**: Supports different version increments based on branch type (feature/*, fix/*, release/*)
