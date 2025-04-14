Here's a complete guide to **create local repositories in JFrog Artifactory** for Azure CLI and PowerShell, upload the latest versions, and use them in your GitHub Actions workflow:

---

### **Step 1: Create Local Repositories in JFrog**
#### **For PowerShell (Debian Package)**
1. Go to **Admin → Repositories → Local → New Local Repository**
   - **Package Type**: Debian  
   - **Repository Key**: `debian-powershell`  
   - **Layout**: `simple-default`  
   - **Checksum Policy**: `verify-if-exists`

#### **For Azure CLI (PyPI Package)**
1. Create another local repository:  
   - **Package Type**: PyPI  
   - **Repository Key**: `pypi-azure-cli`  
   - **Layout**: `simple-default`

---

### **Step 2: Upload Packages to JFrog**
#### **Upload PowerShell (Debian Package)**
```bash
# Download the latest PowerShell .deb package
curl -LO https://packages.microsoft.com/ubuntu/$(lsb_release -rs)/prod/pool/main/p/powershell/powershell_7.4.2-1.deb_amd64.deb

# Upload to JFrog
jf rt upload powershell_*.deb debian-powershell/pool/main/p/powershell/ --flat=false
```

#### **Upload Azure CLI (PyPI Package)**
```bash
# Download Azure CLI wheel
pip download azure-cli --no-deps -d .

# Upload to JFrog
jf rt upload azure_cli-*.whl pypi-azure-cli/
```

---

### **Step 3: Configure GitHub Actions Workflow**
#### **Install PowerShell**
```yaml
steps:
  - name: Add JFrog Debian repo
    run: |
      echo "deb [trusted=yes] https://your-company.jfrog.io/artifactory/debian-powershell $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/jfrog-powershell.list
      sudo apt-get update

  - name: Install PowerShell
    run: sudo apt-get install -y powershell
```

#### **Install Azure CLI**
```yaml
  - name: Configure JFrog PyPI
    run: jf pip-config --server-id-resolve=frigate --repo-resolve=pypi-azure-cli

  - name: Install Azure CLI
    run: |
      pip install azure-cli --user
      echo "$HOME/.local/bin" >> $GITHUB_PATH
```

---

### **Step 4: Verify Installations**
```yaml
  - name: Check versions
    run: |
      pwsh --version
      az --version
```

---

### **Key Advantages**
| Feature | Benefit |
|---------|---------|
| **Controlled Versions** | Pin specific versions in your repo |
| **Air-Gapped Support** | Works offline after initial upload |
| **Security** | Scan packages with JFrog Xray |
| **Performance** | Faster than downloading from public sources |

---

### **Maintenance Tips**
1. **Update Packages**:
   ```bash
   # PowerShell
   curl -LO https://packages.microsoft.com/ubuntu/.../powershell_X.Y.Z.deb
   jf rt upload powershell_*.deb debian-powershell/...

   # Azure CLI
   pip download azure-cli==2.71.0
   jf rt upload azure_cli-*.whl pypi-azure-cli/
   ```

2. **Automate Syncs** (Optional):
   - Set up **Artifactory User Plugins** to auto-download new releases.

Would you like me to provide the exact JFrog REST API commands to automate uploads?
