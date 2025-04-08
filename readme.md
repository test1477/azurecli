Absolutely! Here's the updated and **final version of all 3 files** — with clean handling of the `artifactory_token` passed as an input (not `env`), using `frigate` and `pypi-remote`.

---

## ✅ 1. `.github/actions/azure-cli-setup/scripts/az_cli_setup.sh`

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

---

## ✅ 2. `.github/actions/azure-cli-setup/action.yml`

```yaml
name: 'Azure CLI Setup'
description: 'Omega - Azure CLI installation using Frigate Artifactory'

inputs:
  version:
    description: 'Version of Azure CLI to install'
    required: false
    default: '2.69.0'
  artifactory_token:
    description: 'Frigate Artifactory token'
    required: true

runs:
  using: "composite"
  steps:
    - name: Install JFrog CLI from Frigate
      uses: MorganStanley-Actions/train.setup-jfrog-cli@prod
      with:
        saas-af-token: ${{ inputs.artifactory_token }}
        version: 2.48.0

    - name: Setup Azure CLI via pip from Frigate
      shell: bash
      run: |
        chmod +x scripts/az_cli_setup.sh
        ./scripts/az_cli_setup.sh ${{ inputs.version }}
      working-directory: ${{ github.action_path }}
```

---

## ✅ 3. `.github/workflows/setup-azure-cli.yml`

```yaml
name: Setup Azure CLI via Frigate

on:
  workflow_dispatch:

jobs:
  setup-azure-cli:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Use custom Azure CLI setup action
        uses: ./.github/actions/azure-cli-setup
        with:
          artifactory_token: ${{ secrets.ARTIFACTORY_TOKEN }}
          version: '2.69.0'
```

---

### 🔐 Secret Reminder

Make sure this secret exists:

```
Repo → Settings → Secrets and Variables → Actions → New secret
Name: ARTIFACTORY_TOKEN
Value: <your-frigate-token>
```

---

Let me know if you want to:
- publish this to GitHub Marketplace,
- bundle it into a shared repo for reuse,
- or add validation/logging steps to the shell script.
