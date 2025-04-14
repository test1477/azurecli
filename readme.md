Here's the **corrected approach** for using your custom `tf-providers-powershell` repository in JFrog Artifactory:

---

### **Fixed Workflow for Custom Debian Repository**
```yaml
name: Install PowerShell from tf-providers-powershell

jobs:
  install-powershell:
    runs-on: ubuntu-latest
    steps:
      - name: Add authenticated JFrog repository
        run: |
          # For Ubuntu 20.04 (focal)
          echo "deb [trusted=yes] https://${{ secrets.JFROG_USER }}:${{ secrets.JFROG_ACCESS_TOKEN }}@frigate.jfrog.io/artifactory/tf-providers-powershell focal main" | \
            sudo tee /etc/apt/sources.list.d/jfrog-powershell.list
          
          sudo apt-get update

      - name: Install PowerShell
        run: sudo apt-get install -y powershell

      - name: Verify installation
        run: pwsh --version
```

---

### **Key Requirements**
1. **Repository Structure Must Match**:
   ```
   tf-providers-powershell/
   ├── dists/
   │   └── focal/          # Distribution codename (e.g., focal for Ubuntu 20.04)
   │       └── main/       # Component
   │           └── binary-amd64/
   │               ├── Packages
   │               └── Packages.gz
   └── pool/
       └── main/
           └── p/
               └── powershell/
                   └── powershell_7.4.2-1.deb
   ```

2. **Authentication**:
   - Credentials are embedded in the URL (`user:token@domain`)
   - Alternative: Use `/etc/apt/auth.conf` for better security

---

### **Troubleshooting 403 Errors**
If you still get **403 Forbidden**, verify:

1. **Credentials**:
   ```bash
   # Test repository access
   curl -u ${{ secrets.JFROG_USER }}:${{ secrets.JFROG_ACCESS_TOKEN }} \
     https://frigate.jfrog.io/artifactory/tf-providers-powershell/dists/focal/main/binary-amd64/Packages
   ```

2. **Permissions**:
   - Ensure the GitHub Actions user has **read** access to:
     - `tf-providers-powershell`
     - All parent folders

3. **Repository Layout**:
   ```bash
   # Check if Packages.gz exists
   curl -I https://frigate.jfrog.io/artifactory/tf-providers-powershell/dists/focal/main/binary-amd64/Packages.gz
   ```

---

### **Alternative (More Secure) Authentication**
```yaml
- name: Configure APT auth
  run: |
    echo "machine frigate.jfrog.io
    login ${{ secrets.JFROG_USER }}
    password ${{ secrets.JFROG_ACCESS_TOKEN }}" | sudo tee /etc/apt/auth.conf.d/jfrog.conf
    sudo chmod 600 /etc/apt/auth.conf.d/jfrog.conf

- name: Add repository (no credentials in URL)
  run: |
    echo "deb https://frigate.jfrog.io/artifactory/tf-providers-powershell focal main" | \
      sudo tee /etc/apt/sources.list.d/jfrog-powershell.list
```

---

### **Final Checks**
1. **Verify Package Availability**:
   ```bash
   sudo apt-cache policy powershell
   ```

2. **Manual Download Test**:
   ```bash
   curl -u $USER:$TOKEN -LO https://frigate.jfrog.io/artifactory/tf-providers-powershell/pool/main/p/powershell/powershell_7.4.2-1.deb
   ```

This matches your documented approach while ensuring proper authentication and repository structure. The key is maintaining the exact Debian repository layout in your `tf-providers-powershell` repo.
