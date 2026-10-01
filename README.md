---
kind: reference
description: Reusable Windows installer and frameless Tauri shell for supported upstream Web applications.
---

# Portable Web Shell

English | [中文](README.zh.md)

## Summary

Portable Web Shell combines a reusable PowerShell installer with a compiled frameless Tauri/Rust window. Project recipes select the upstream repository, branch, build commands, and readiness announcement. Application sources are downloaded into a separate installation; the installer does not patch their UI or runtime code. The same shell executable survives application updates. DeepSeek Harness is the included, verified application recipe.

## Table of Contents

- [How it works](#how-it-works)
- [Install and run](#install-and-run)
- [Update and move](#update-and-move)
- [Project recipes](#project-recipes)
- [Files and requirements](#files-and-requirements)
- [Startup and shutdown](#startup-and-shutdown)
- [Verification and recovery](#verification-and-recovery)
- [Build and distribution](#build-and-distribution)
- [Dev Note](#dev-note)

<a id="how-it-works"></a>

## How it works

The reusable parts are the installer scripts and compiled shell. Each application needs a recipe describing its actual installation and launch commands. The current installer supports public GitHub source archives with a root `package.json` declaring an exact pnpm version. Another runtime, private repository, or different readiness protocol requires an explicit adapter. Changing a repository name alone does not make an arbitrary application compatible.

| Action | Version behavior |
| --- | --- |
| Download the shell ZIP | Obtains the installer and shell; it contains no installed upstream app. |
| `Install.cmd` or `Update.cmd` | Resolves the configured upstream branch at that moment, downloads its exact commit, builds and verifies it, and activates it only on success. |
| Desktop shortcut or `Run.cmd` | Starts the already installed revision. It does not check for updates or download an application. |
| Install with a tag or commit in `Ref` | Uses that selected revision instead of following the latest branch. |

For DeepSeek, the tracked branch is official upstream `master`. Its latest commit can be newer than its latest published release. The selected commit is recorded in `installed.json`; commits arriving after resolution are picked up on the next Install or Update. If upstream does not build or pass the startup check, the installer reports failure and retains the previous active launch configuration.

```mermaid
flowchart LR
  Recipe["Application recipe"] --> Installer["Install or Update"]
  Upstream["Current configured upstream reference"] --> Installer
  Installer --> App["Download, build and verify exact commit"]
  App --> Shortcut["Create desktop shortcut"]
  Shortcut --> Shell["Persistent frameless Tauri executable"]
  Shell --> UI["Original application Web UI"]
```

The shell starts the application's local server, reads its announced loopback URL, and loads that original page in its own WebView. Its window controls are a separate local document. Application updates reuse the shell executable; shell implementation updates are distributed separately in this repository's releases.

<a id="install-and-run"></a>

## Install and run

Download the compiled Windows x64 ZIP from [Releases](https://github.com/walladanger/portable-web-shell/releases), extract it into a permanent folder, and run `Install.cmd`. It includes `bin/portable-web-shell.exe`; you can also build that executable from this source checkout using [Build-Shell.ps1](Build-Shell.ps1). The bundle can live independently of this repository checkout.

| File | Action |
| --- | --- |
| [Install.cmd](Install.cmd) | Download a contained Node runtime, resolve the upstream branch to a commit, download its source archive, install dependencies, build, verify the Web UI, activate the revision, and create a desktop shortcut. |
| [Run.cmd](Run.cmd) | Launch the installed application in the frameless window. |
| [Update.cmd](Update.cmd) | Prepare and verify the current upstream revision using the same installer. |
| [Repair-Shortcut.cmd](Repair-Shortcut.cmd) | Recreate the desktop shortcut for this installation's current path. |

The default recipe is [projects/deepseek.json](projects/deepseek.json), pointing to `deepseek-ai/deepseek-harness` and its `master` branch. The default installation folder is `installed/deepseek` beside these scripts. Downloading a newer revision does not require merging a fork or recreating wrapper source.

The [Manage.ps1](Manage.ps1) `Plan` action displays the selected recipe and destination without installing anything. Its `InstallRoot` parameter selects another destination, `Ref` selects an upstream branch/tag/commit, and `NoShortcut` suppresses shortcut creation. The command files forward these parameters to the script.

<a id="update-and-move"></a>

## Update and move

Updates download each source revision into `app/versions/<commit>`, build it, and test its authenticated Web entry using a separate verification home. `launch.json` is replaced only after that test succeeds. Earlier revisions remain in the folder; they are not automatically deleted or selected as a downgrade. DeepSeek's persisted data has its own compatibility rules, so retaining an older build does not establish that it can read newer sessions.

Close the application before updating. An installation lock rejects simultaneous installers. The recorded upstream commit and runtime versions are in `installed.json`.

Close the application and move the installer bundle and its `installed` folder together. After changing the installation path or machine, run `Install.cmd` again: pnpm dependencies can contain Windows junctions that need relinking, and the shortcut contains an absolute destination. The installer retains application data, moves the previous generated dependency directory into `data/dependency-backups` when the recorded root changed, rebuilds dependencies and verifies the app, and recreates the shortcut. Shortcut repair alone does not reinstall dependencies. Paths to workspaces outside the installation may need to be selected again on another computer.

<a id="project-recipes"></a>

## Project recipes

The shell reads an installation's `launch.json`; it is independent of the DeepSeek repository and can launch another contained native executable. The current installer adapter supports GitHub source archives with a root `package.json` declaring an exact pnpm version. Other build systems require an installer adapter rather than pretending that a pnpm recipe applies to them.

| Recipe field | Meaning |
| --- | --- |
| `schemaVersion` | Supported recipe version, currently `1`. |
| `id`, `name` | Installation subfolder identifier and displayed application/shortcut name. |
| `shortcutName` | Optional desktop shortcut name; defaults to `name`. DeepSeek uses `DeepSeek Harness (Frameless)`. |
| `repository`, `ref` | GitHub repository and upstream reference to resolve. |
| `nodeMajor` | Portable Windows x64 Node release line. |
| `buildTools` | Required boolean: `true` checks for Microsoft C++ tools and offers installation when missing; `false` skips that check. |
| `installCommands` | Arrays of arguments passed to the contained pnpm executable. Each command must succeed. |
| `launchArguments` | Arguments forwarded through pnpm when the shell starts the application. |
| `readinessPrefix` | Prefix of the newline-terminated stdout record containing the app's loopback HTTP URL. |
| `requireToken` | Require DeepSeek's root URL with exactly one nonempty `token` query parameter. |

The DeepSeek recipe runs its supported `dsh web` profile with `--no-open` and port `0`, allowing Windows to allocate a free port. The actual URL comes from DeepSeek's own announcement. A recipe for another application must match that application's real install and startup behavior.

For another compatible project, copy the DeepSeek recipe into a new file in `projects/`, choose a distinct `id` and shortcut name, and supply its real repository, branch, Node major, pnpm build/start commands, and stdout URL prefix. Test installation, authenticated page loading when applicable, and shutdown before treating that recipe as supported. Pass the selected recipe to the existing command file, for example `Install.cmd -Project projects/my-project.json`; the application installs under `installed/<id>`. That example only works after the recipe has been written for the actual project.

<a id="files-and-requirements"></a>

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

Microsoft WebView2 is a Windows runtime prerequisite. If it is missing, the installer downloads and runs Microsoft's bootstrapper. Its shared runtime belongs to Windows; application WebView profiles and caches belong to the installation's `data` folder. A source build of the shell requires Rust and Microsoft C++ Build Tools; [Build-Shell.ps1](Build-Shell.ps1) offers their installation or official download when missing.

The desktop shortcut is necessarily outside the folder. Dependencies required by later DeepSeek tools or plugins may have additional system requirements. User-selected workspaces can also live outside this installation. Runtime data, credentials and downloaded builds are ignored by Git.

Opening the Web UI does not require a model API request. To run an agent session, configure the selected model provider's account or API credential through DeepSeek's own configuration UI, and provide network access to that provider. The default local credential store writes beneath `DSH_HOME`, here `data/dsh-home`; credential configuration is separate from the temporary loopback launch token. The wrapper does not supply a model key or install a local language model. Tools enabled in DeepSeek may also need their own programs, credentials, and workspace permissions.

<a id="startup-and-shutdown"></a>

## Startup and shutdown

```mermaid
flowchart TD
  Recipe["Project recipe"] --> Install["Reusable installer"]
  Upstream["Official DeepSeek master"] --> Install
  Install --> Revision["Downloaded and built commit"]
  Install --> Check["Isolated authenticated HTTP check"]
  Check --> Active["Activate launch.json"]
  Active --> Shortcut["Desktop shortcut"]
  Shortcut --> Shell["Persistent frameless Tauri shell"]
  Shell --> Chrome["Local drag strip and window controls"]
  Shell --> Backend["Contained Node: dsh web"]
  Backend --> Original["Original Web UI in its own WebView"]
  Shell --> Data["Contained data, profiles and logs"]
  Backend --> Data
```

The window has a local title strip with drag, minimize, maximize/restore, and close controls. The app loads in a separate child WebView. No titlebar HTML, JavaScript, or CSS is inserted into DeepSeek's document. Only the local chrome can invoke native window actions; the application view receives no Tauri permissions. Each installation has its own single-instance identifier.

```mermaid
sequenceDiagram
  participant User
  participant Chrome as Local window controls
  participant Shell as Tauri shell
  participant App as Original application WebView
  participant Server as DeepSeek Web server
  User->>Shell: Open desktop shortcut
  Shell->>Server: Start contained dsh web
  Server-->>Shell: Announce authenticated loopback URL
  Shell->>App: Navigate to that URL
  App->>Server: Request index with launch token
  Server-->>App: Authentication cookie and redirect
  App->>Server: Request index with cookie
  Server-->>App: Original HTML and application assets
  User->>Chrome: Close window
  Chrome->>Shell: Local native command
  Shell->>Server: Stop owned backend process tree
```

The Windows backend is spawned suspended, assigned to an owned Windows Job, and resumed after assignment. Closing or exiting the shell releases that Job and terminates its backend tree. This is forced termination, not DeepSeek's graceful shutdown protocol. Startup timeouts and process exits also stop the owned backend. Backend failure after readiness is reported in the title strip.

The shell accepts loopback HTTP readiness URLs, rejects foreign origins and malformed token URLs, and opens external HTTP(S) citations or authorization links through Windows' URL handler. Launch URL queries are redacted in the captured stdout log. Ambient environment variables whose names contain KEY, SECRET, TOKEN or PASSWORD are removed from the backend environment; configure DeepSeek credentials through its original UI and contained Harness home.

```mermaid
flowchart LR
  User["Session input in original Web UI"] --> Backend["Local Harness backend"]
  Backend --> Provider["Configured remote model provider"]
  Provider --> Backend
  Backend --> Tools["Enabled tools and selected workspace"]
  Tools --> Backend
  Backend --> Store["Contained sessions, settings and credentials"]
  Backend --> Events["Results and session updates"]
  Events --> User
```

The shell manages the window and backend lifetime. DeepSeek owns session execution, provider requests, tool selection and persistence. The Web UI submits user actions to that local backend and displays its results; the model provider connection originates from Harness. Downloading an upstream revision updates those application components together, while the wrapper remains a separate executable.

<a id="verification-and-recovery"></a>

## Verification and recovery

The installer uses the shell's `--smoke` mode to launch the real backend, exchange its token for a cookie, request the HTML Web UI, and stop the backend. Installation failures leave the previous active `launch.json` in place. This checks startup and the authenticated index, not every DeepSeek feature or API provider.

[Test-Wrapper.ps1](Test-Wrapper.ps1) exercises the compiled shell using real HTTP and descendant Node processes. It covers fragmented stdout readiness, authenticated redirects, unauthorized responses, startup timeout, parent exit, descendant cleanup and launch-token log redaction. Rust tests cover readiness URL and owned-path validation.

[Test-Install.ps1](Test-Install.ps1) checks long-path extraction, rejection of unsafe ZIP entries, diagnostic capture, and native exit codes under Windows PowerShell. [Test-Window.ps1](Test-Window.ps1) checks original page rendering and native window controls on an interactive Windows desktop.

If installation or startup fails, inspect the newest file in `installed/deepseek/data/logs`. Installer scripts stop on nonzero command exit codes. A startup error also appears in the title strip; invalid launch files produce a native error dialog. Retry installation after correcting the reported problem. The `Verify` action runs the active configuration's HTTP check without opening the GUI.

<a id="build-and-distribution"></a>

## Build and distribution

The [Windows workflow](https://github.com/walladanger/portable-web-shell/actions/workflows/windows.yml) builds the reusable executable on Windows, runs Rust and executable behavior checks, and publishes a ZIP artifact containing the executable, scripts, recipes, licenses, and README. It does not install DeepSeek in CI or publish an app release automatically.

The bundle deliberately excludes installed applications, logs, credentials, dependency caches and Rust build output. The shell is built once for distribution; each user installation then downloads its upstream app. Update the shell separately when its own implementation changes.

[Package.ps1](Package.ps1) creates that curated ZIP locally. The wrapper executable is unsigned; the workflow does not use a signing certificate. The wrapper is a separate artifact from DeepSeek's signed official desktop client.

<a id="dev-note"></a>

## Dev Note

This installer and shell are outside DeepSeek's pnpm workspace. They launch the existing `dsh` profile and introduce no Harness package, plugin, model-visible event, or session-format change. Their HTTP and Windows lifecycle evidence is owned here rather than by a recorded Harness conversation.

This standalone repository owns future shell development. [SOURCE.md](SOURCE.md) records the original extraction and attribution; its Git history does not include the DeepSeek application repository.
