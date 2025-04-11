▼Run jf dotnet tool install -g PowerShell --version 7.3.6

jf dotnet tool install -g PowerShell --version 7.3.6

echo "$HOME/.dotnet/tools" >> $GITHUB_PATH

6 shell: /usr/bin/bash -e {0}

7

env:

ARTIFACTORY_TOKEN: ***

JFROG_CLI_ENV_EXCLUDE: *password*; *pass*;*key*;*secret*;*token*

JFROG_CLI_OFFER_CONFIG: false

JFROG_CLI_BUILD_NAME: Setup powershell

JFROG_CLI_BUILD_NUMBER: 3

JFROG_CLI_BUILD_URL: https://github.com/Eaton-Vance-Corp/octopus yml/actions/runs/14408490732

JFROG_CLI_USER_AGENT: setup-jfrog-cli-github-action/4.0.0

pythonLocation: /home/ubuntu/actions-runner/_work/_tool/Python/3.9.22/x64

PKG_CONFIG_PATH: /home/ubuntu/actions-runner/_work/_tool/Python/3.9.22/x64/lib/pkgconfig

9

10

11

12

13

14

15

16

17 Python_ROOT_DIR: /home/ubuntu/actions-runner/_work/_tool/Python/3.9.22/x64

18

19

20

21

22

Python2_ROOT_DIR: /home/ubuntu/actions-runner/_work/_tool/Python/3.9.22/x64

Python3_ROOT_DIR: /home/ubuntu/actions-runner/_work/_tool/Python/3.9.22/x64

LD_LIBRARY_PATH: /home/ubuntu/actions-runner/_work/_tool/Python/3.9.22/x64/lib

3FROG_CLI_LOG_LEVEL: INFO

PWSH_VERSION: 7.3.6

23 17:15:10 [Info] Running dotnet...

24 Error: 0 [Error] exec: "dotnet": executable file not found in $PATH

25 Error: Process completed with exit code 1.
