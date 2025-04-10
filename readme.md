# Installing PowerShell via GitHub Actions with JFrog Artifactory

To install PowerShell in a GitHub Actions workflow while leveraging JFrog Artifactory for package management, follow this comprehensive guide:

## Method 1: Using JFrog Artifactory (Recommended for Enterprise)

```yaml
steps:
  - name: Install JFrog CLI
    uses: jfrog/setup-jfrog-cli@v2

  - name: Configure JFrog Artifactory
    run: |
      jf config add my-artifactory \
        --url=https://your-company.jfrog.io/artifactory \
        --access-token=${{ secrets.JFROG_ACCESS_TOKEN }}

  - name: Install PowerShell via JFrog
    run: |
      # For Linux (Ubuntu/Debian)
      jf rt download debian-remote/pool/main/p/powershell/ --flat
      sudo dpkg -i powershell*.deb
      
      # For Windows
      jf rt download generic-remote/powershell/ -flat
      msiexec.exe /i powershell*.msi /quiet
      
      # Verify installation
      pwsh --version
```

## Method 2: Direct Installation (Simpler Approach)

```yaml
steps:
  - name: Install PowerShell (Linux)
    if: runner.os == 'Linux'
    run: |
      # Download from Microsoft's official repo
      wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
      sudo dpkg -i packages-microsoft-prod.deb
      sudo apt-get update
      sudo apt-get install -y powershell
      pwsh --version

  - name: Install PowerShell (Windows)
    if: runner.os == 'Windows'
    run: |
      winget install --id Microsoft.PowerShell --accept-package-agreements --accept-source-agreements
      pwsh --version

  - name: Install PowerShell (macOS)
    if: runner.os == 'macOS'
    run: |
      brew install --cask powershell
      pwsh --version
```

## Method 3: Using JFrog with PowerShell Gallery

```yaml
steps:
  - name: Set up JFrog NuGet repository
    run: |
      jf dotnet-config --server-id-resolve=my-artifactory --repo-resolve=nuget-remote

  - name: Install PowerShell via JFrog NuGet
    run: |
      # Install PowerShell Core via .NET Global Tool
      dotnet tool install --global PowerShell --version 7.3.4
      pwsh --version
```

## Verification Step (All Methods)

```yaml
  - name: Verify PowerShell installation
    run: |
      pwsh --version
      pwsh -c "Write-Host 'PowerShell is working!'"
```

## Key Considerations:

1. **JFrog Configuration**:
   - Ensure your Artifactory instance has the appropriate repositories configured (Debian for Linux packages, NuGet for .NET tools)
   - Set up proper security permissions for your GitHub Actions runner

2. **Platform Support**:
   - PowerShell is available for Windows, Linux, and macOS
   - Installation methods vary by platform

3. **Version Pinning**:
   - For production use, pin to specific versions
   - Example: `sudo apt-get install -y powershell=7.3.4-1.ubuntu.22.04`

4. **Proxy Settings** (if needed):
   ```yaml
   env:
     HTTP_PROXY: http://your-proxy:port
     HTTPS_PROXY: http://your-proxy:port
   ```

For enterprise environments, Method 1 is recommended as it provides:
- Artifact traceability
- Security scanning through JFrog Xray
- Better performance through caching
- Compliance with internal policies

Would you like me to tailor this further for your specific JFrog repository structure or security requirements?
