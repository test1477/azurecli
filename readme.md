To implement the `setup-azure-cli` functionality you described using GitHub Actions and Azure CLI, you can create two components: one for the Bash script (`az_cli_setup.sh`) and one for the `action.yml` file to define the steps in the GitHub Action workflow.

### Step 1: Bash Script (`az_cli_setup.sh`)

The Bash script is responsible for installing JFrog CLI and the Azure CLI with the given version. You can use the following script template, which aligns with your example:

#### `scripts/az_cli_setup.sh`:

```bash
#!/bin/bash

set -e

# Retrieve the Azure CLI version passed as argument
AZ_CLI_VERSION=${1}

# Install JFrog CLI if not already installed
jf pip-config --server-id-resolve msartaz --repo-resolve pypi repo

# Upgrade pip
jf pip install pip --upgrade --no-warn-script-location --no-compile --disable-pip-version-check

# Install Azure CLI
jf pip install azure-cli==$AZ_CLI_VERSION --no-warn-script-location --no-compile --disable-pip-version-check

# Add Azure CLI to the system PATH
echo "$HOME/.local/bin" >> $GITHUB_PATH
```

### Step 2: `action.yml` for GitHub Actions

The `action.yml` file defines the parameters (such as the Azure CLI version and Artifactory token) and sets up the necessary steps to run the Bash script within a GitHub Actions workflow.

#### `action.yml`:

```yaml
name: 'Azure-cli Setup'
description: 'Setup Azure CLI in your GitHub Actions environment'
inputs:
  version:
    description: 'Version of Azure CLI to install'
    required: false
    default: '2.66.0'
  art_token:
    description: 'Artifactory token'
    required: true

runs:
  using: "composite"
  steps:
    - name: Install JFrog CLI
      uses: eaton-vance/setup-jfrog-cli@v4

    - name: Setup Azure CLI
      run: |
        chmod +x scripts/az_cli_setup.sh
        ./scripts/az_cli_setup.sh ${{ inputs.version }}
      shell: bash
      working-directory: ${{ github.action_path }}
```

### Step 3: GitHub Actions Workflow (Calling the Setup Action)

In your GitHub Actions workflow file, you can use this custom action (`setup-azure-cli`) and call it to install the Azure CLI with the version specified by the input.

#### Example `deploy_qa.yml`:

```yaml
name: deploy QA

permissions:
  contents: write
  actions: write
  id-token: write

on:
  workflow_call:

env:
  ARTIFACTORY_TOKEN: ${{ secrets.ARTIFACTORY_TOKEN }}

jobs:
  deploy-arm-qa:
    runs-on: azure-nonprod
    environment:
      name: qa
    outputs:
      release_number: ${{ steps.setup_release_number.outputs.TRAIN_RELEASE }}

    steps:
      - name: Check out the repo
        uses: actions/checkout@v4

      - name: Install JFrog CLI
        uses: eaton-vance/setup-jfrog-cli@v4
        with:
          version: 2.25.2
          jfrog_token: ${{ secrets.ARTIFACTORY_TOKEN }}

      - name: Setup Azure CLI
        uses: ./path-to-your-action # Reference to your setup-azure-cli action
        with:
          version: 2.66.0
          art_token: ${{ secrets.ARTIFACTORY_TOKEN }}

      - name: Python Check
        run: python -V && python -m pip -V
        continue-on-error: true
```

### Explanation:

1. **`az_cli_setup.sh`**: A Bash script that installs both JFrog CLI and Azure CLI with the specified version. It also ensures the necessary dependencies are updated (like pip).

2. **`action.yml`**: A GitHub Action definition that provides inputs such as the version of the Azure CLI and Artifactory token. It then runs the Bash script to install the Azure CLI.

3. **GitHub Actions Workflow (`deploy_qa.yml`)**: A job that uses the custom action `setup-azure-cli` to install the Azure CLI and performs other tasks (such as Python version checks). The workflow also integrates with JFrog CLI.

Make sure that:
- The Artifactory token is stored in GitHub Secrets.
- The path to your custom action (`setup-azure-cli`) is correct.

This setup should automate the installation of the Azure CLI as part of your deployment pipeline.
