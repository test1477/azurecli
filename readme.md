The error occurs because the **.NET SDK is not installed** on your runner, which is required for `dotnet tool install`. Here's how to fix this for your **RHEL 7.9** system:

---

### **Solution 1: Install .NET SDK First (Recommended)**
Add these steps **before** the PowerShell installation:

```yaml
- name: Install .NET 6.0 SDK (Required for dotnet tool)
  run: |
    sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm
    sudo yum install -y dotnet-sdk-6.0
    echo "export PATH=$PATH:$HOME/.dotnet" >> $HOME/.bashrc
    source $HOME/.bashrc

- name: Verify .NET installation
  run: dotnet --version
```

Then proceed with your existing PowerShell install:
```yaml
- name: Install PowerShell via JFrog NuGet
  run: |
    jf dotnet-config --server-id-resolve=frigate --repo-resolve=nuget-remote
    jf dotnet tool install -g PowerShell --version $PWSH_VERSION
    echo "$HOME/.dotnet/tools" >> $GITHUB_PATH
```

---

### **Solution 2: Alternative (No .NET Required) - Use Official RPM**
If you prefer to avoid .NET dependencies:
```yaml
- name: Download and Install PowerShell RPM
  run: |
    curl -LO https://github.com/PowerShell/PowerShell/releases/download/v$PWSH_VERSION/powershell-$PWSH_VERSION-1.rhel.7.x86_64.rpm
    sudo yum install -y ./powershell-$PWSH_VERSION-1.rhel.7.x86_64.rpm
    pwsh --version
  env:
    PWSH_VERSION: 7.3.6
```

---

### **Key Fixes**
| Issue | Solution |
|-------|----------|
| `dotnet: not found` | Install .NET 6.0 SDK first |
| RHEL 7.9 compatibility | Use Microsoft's RPM package |
| JFrog integration | Configure `nuget-remote` for .NET tools |

---

### **Full Corrected Workflow**
```yaml
steps:
  - name: Install .NET 6.0 SDK
    run: |
      sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm
      sudo yum install -y dotnet-sdk-6.0
      echo "$HOME/.dotnet" >> $GITHUB_PATH

  - name: Configure JFrog NuGet
    run: jf dotnet-config --server-id-resolve=frigate --repo-resolve=nuget-remote

  - name: Install PowerShell
    run: |
      jf dotnet tool install -g PowerShell --version $PWSH_VERSION
      echo "$HOME/.dotnet/tools" >> $GITHUB_PATH
    env:
      PWSH_VERSION: 7.3.6

  - name: Verify
    run: pwsh --version
```

---

### **Why This Works**
1. **.NET SDK** provides the `dotnet` command-line tool
2. **JFrog NuGet** configuration allows secure package resolution
3. **Global tool install** places `pwsh` in `$HOME/.dotnet/tools`

For **enterprise environments**, ensure:
- Your `nuget-remote` repository proxies `https://api.nuget.org/v3/index.json`
- The JFrog service account has read permissions

Would you like me to provide the JFrog admin-side configuration for NuGet repositories?
