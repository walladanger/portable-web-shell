## Files and requirements

```text
portable-web-shell/
  Install.cmd / Update.cmd / Run.cmd / Repair-Shortcut.cmd
  Manage.ps1
  projects/deepseek.json
  bin/portable-web-shell.exe
  installed/deepseek/
    bin/portable-web-shell.exe
    runtime/node/                    contained Node and npm
    runtime/pnpm-<version>/          contained package manager
    app/versions/<commit>/          original source and generated build files
    launch.json                     active relative-path launch configuration
    installed.json                  upstream commit and runtime version record
    data/
      dsh-home/                     DeepSeek profiles, credentials, sessions and settings
      agents-home/                  DeepSeek agent files
      logs/                         installer and backend diagnostics
      webview-chrome/                shell WebView profile
      webview-app/                   application WebView profile
      npm-cache/ and pnpm-store/     package caches
      temp/                         downloaded archives and temporary files
      verification/                 isolated home used before activation
```

Installed users do not need Rust or Visual Studio to run a prebuilt shell. Windows PowerShell 5.1 or later and network access are required for installation. Node and pnpm are downloaded into the folder. Installation and launch also redirect the process home, AppData, Electron cache, and browser download caches into it. Node's ZIP is checked against the official release checksum. DeepSeek source is fetched by the exact commit returned by GitHub.

Microsoft WebView2 is a Windows runtime prerequisite. If it is missing, the installer downloads and runs Microsoft's bootstrapper. Its shared runtime belongs to Windows; application WebView profiles and caches belong to the installation's `data` folder. A source build of the shell requires Rust and Microsoft C++ Build Tools; [Build-Shell.ps1](../Build-Shell.ps1) offers their installation or official download when missing.

The desktop shortcut is necessarily outside the folder. Dependencies required by later DeepSeek tools or plugins may have additional system requirements. User-selected workspaces can also live outside this installation. Runtime data, credentials and downloaded builds are ignored by Git.

Opening the Web UI does not require a model API request. To run an agent session, configure the selected model provider's account or API credential through DeepSeek's own configuration UI, and provide network access to that provider. The default local credential store writes beneath `DSH_HOME`, here `data/dsh-home`; credential configuration is separate from the temporary loopback launch token. The wrapper does not supply a model key or install a local language model. Tools enabled in DeepSeek may also need their own programs, credentials, and workspace permissions.
