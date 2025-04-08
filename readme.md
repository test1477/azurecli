Absolutely! Here's the **full and correct setup** including:

1. ✅ The **Bash script** (`az_cli_setup.sh`)
2. ✅ The **Composite GitHub Action** (`action.yml`)
3. ✅ The **GitHub Workflow file** (`setup-azure-cli.yml`)

---

## 🧩 1. `scripts/az_cli_setup.sh`

```bash
#!/bin/bash
set -e

AZ_CLI_VERSION=${1}

# Configure pip to use Frigate Artifactory (pypi-remote repo)
jf pip-config --server-id-resolve frigate --repo-resolve pypi-remote

# Upgrade pip using Artifactory
jf pip install pip --upgrade --no-warn-script-location --no-compile --disable-pip-version-check

# Install the specified version of Azure CLI
jf pip install azure-cli==$AZ_CLI_VERSION --no-warn-script-location --no-compile --disable-pip-version-check

# Add local bin to GitHub Actions PATH
echo "$HOME/.local/bin" >> "$GITHUB_PATH"
```

> Save this script at: `.github/actions/azure-cli-setup/scripts/az_cli_setup.sh`

---

## 🧩 2. `.github/actions/azure-cli-setup/action.yml`

```yaml
name: 'Azure CLI Setup'
description: 'Omega - Azure CLI installation using Frigate Artifactory'

inputs:
  version:
    description: 'Version of Azure CLI to install'
    required: false
    default: '2.69.0'
  art_token:
    description: 'Frigate Artifactory token'
    required: true

runs:
  using: "composite"
  steps:
    - name: Install JFrog CLI from Frigate
      uses: MorganStanley-Actions/train.setup-jfrog-cli@prod
      with:
        saas-af-token: ${{ inputs.art_token }}
        version: 2.48.0

    - name: Setup Azure CLI via pip from Frigate
      shell: bash
      run: |
        chmod +x scripts/az_cli_setup.sh
        ./scripts/az_cli_setup.sh ${{ inputs.version }}
      working-directory: ${{ github.action_path }}
```

---

## 🧩 3. GitHub Workflow: `.github/workflows/setup-azure-cli.yml`

```yaml
name: Setup Azure CLI via Frigate

on:
  workflow_dispatch:

jobs:
  setup-azure-cli:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v3

      - name: Use custom Azure CLI setup
        uses: ./.github/actions/azure-cli-setup
        with:
          art_token: ${{ secrets.ARTIFACTORY_TOKEN }}
          version: '2.69.0'
```

> 🔐 Make sure to set `ARTIFACTORY_TOKEN` in your repo's **Secrets**:
>
> `Settings > Secrets and variables > Actions > New repository secret`

---

### ✅ Directory Layout

```
.github/
│
├── actions/
│   └── azure-cli-setup/
│       ├── action.yml
│       └── scripts/
│           └── az_cli_setup.sh
│
└── workflows/
    └── setup-azure-cli.yml
```

---

Let me know if you also want:
- A way to test Azure CLI install success (`az --version`)
- To add caching or fallback if Artifactory is unavailable
- Support for installing extensions like `azure-devops` or `aks` via CLI

Happy to help you extend it!
