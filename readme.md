Since your GitHub Actions runner is using **Ubuntu** (not RHEL as previously thought), here's the corrected approach to install PowerShell using `apt-get` while still leveraging your JFrog Artifactory:

---

### **Corrected Ubuntu Installation via JFrog**
```yaml
steps:
  - name: Install prerequisites
    run: sudo apt-get update && sudo apt-get install -y wget apt-transport-https

  - name: Configure JFrog for Debian packages
    run: |
      # Add JFrog as a trusted source (if hosting Debian packages)
      echo "deb [trusted=yes] https://frigate.jfrog.io/artifactory/debian-remote $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/jfrog.list
      sudo apt-get update

  - name: Install PowerShell (from JFrog or Microsoft)
    run: |
      # Try JFrog first, fallback to Microsoft if not found
      if ! sudo apt-get install -y powershell; then
        echo "Installing from Microsoft..."
        wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
        sudo dpkg -i packages-microsoft-prod.deb
        sudo apt-get update
        sudo apt-get install -y powershell
      fi
      pwsh --version
```

---

### **Alternative: Python Package via JFrog PyPI**
If you prefer using your existing PyPI remote:
```yaml
steps:
  - name: Configure JFrog PyPI
    run: jf pip-config --server-id-resolve=frigate --repo-resolve=pypi-remote

  - name: Install PowerShell Python package
    run: |
      python -m pip install powershell --user
      echo "$HOME/.local/bin" >> $GITHUB_PATH
      pwsh --version
```

---

### **Key Changes from Original Approach**
| Issue | RHEL Solution | Ubuntu Correction |
|-------|--------------|------------------|
| Package Manager | `yum`/`rpm` | `apt-get`/`deb` |
| Install Method | RPM package | Debian package |
| JFrog Config | NuGet repo | Debian repo or PyPI |
| Dependencies | `libicu` | `apt-transport-https` |

---

### **Verification**
```yaml
- name: Verify installation
  run: |
    pwsh --version
    # Expected output: PowerShell 7.3.6
```

---

### **Troubleshooting Ubuntu Issues**
1. **If JFrog doesn't have Ubuntu packages**:
   ```bash
   # Fallback to Microsoft's repo
   curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
   sudo apt-add-repository "deb [arch=amd64] https://packages.microsoft.com/ubuntu/$(lsb_release -rs)/prod $(lsb_release -cs) main"
   sudo apt-get install -y powershell
   ```

2. **Permission issues**:
   ```bash
   sudo chown -R $USER:$USER /usr/local/share/powershell
   ```

---

### **Recommended Approach**
For Ubuntu runners:
1. **Use Microsoft's official Debian packages** (most reliable)
2. **Only use JFrog PyPI** if you need Python integration
3. **Avoid .NET global tools** on Ubuntu unless already configured

Would you like me to provide the JFrog admin steps to configure a Debian repository for Ubuntu packages?
