Perfect — thanks for confirming! So your setup looks like this:

- ✅ **Artifactory Server ID**: `frigate`
- ✅ **PyPI Repository in Artifactory**: `pypi-remote`
- ✅ **Client**: `pip` (via `jf pip install`)
- ✅ **Org**: `omega` (for context, but not used in script)

---

### ✅ Final Bash Script: `az_cli_setup.sh`

```bash
#!/bin/bash
set -e

AZ_CLI_VERSION=${1}

# Configure pip to resolve from Artifactory (Frigate instance)
jf pip-config --server-id-resolve frigate --repo-resolve pypi-remote

# Upgrade pip using Frigate's PyPI mirror
jf pip install pip --upgrade --no-warn-script-location --no-compile --disable-pip-version-check

# Install Azure CLI from Artifactory using pip
jf pip install azure-cli==$AZ_CLI_VERSION --no-warn-script-location --no-compile --disable-pip-version-check

# Ensure ~/.local/bin is added to GitHub Actions PATH
echo "$HOME/.local/bin" >> "$GITHUB_PATH"
```

---

### ✅ `action.yml` for GitHub Composite Action

No changes needed here if you already have:

```yaml
name: 'Azure-cli Setup'
description: 'Omega - Azure CLI Setup via Frigate Artifactory'

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
    - name: Install JFrog CLI (Frigate)
      uses: MorganStanley-Actions/train.setup-jfrog-cli@prod
      with:
        saas-af-token: ${{ inputs.art_token }}
        version: 2.48.0

    - name: Setup Azure CLI from Frigate
      shell: bash
      run: |
        chmod +x scripts/az_cli_setup.sh
        ./scripts/az_cli_setup.sh ${{ inputs.version }}
      working-directory: ${{ github.action_path }}
```

---

### ✅ Summary

- You're using **JFrog CLI (Frigate)** to install Azure CLI via **pip**.
- Packages are pulled from **`pypi-remote`** repo.
- Fully automatable in **GitHub Actions**, works inside `runner` environment (e.g., `ubuntu-latest`).
- Add the Artifactory token in **GitHub Secrets** as `ARTIFACTORY_TOKEN`.

Want me to package this into a repo structure or provide a test workflow to try it out?
