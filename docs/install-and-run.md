## Install and run

Download the compiled Windows x64 ZIP from [Releases](https://github.com/walladanger/portable-web-shell/releases), extract it into a permanent folder, and run `Install.cmd`. It includes `bin/portable-web-shell.exe`; you can also build that executable from this source checkout using [Build-Shell.ps1](../Build-Shell.ps1). The bundle can live independently of this repository checkout.

| File | Action |
| --- | --- |
| [Install.cmd](../Install.cmd) | Download a contained Node runtime, resolve the upstream branch to a commit, download its source archive, install dependencies, build, verify the Web UI, activate the revision, and create a desktop shortcut. |
| [Run.cmd](../Run.cmd) | Launch the installed application in the frameless window. |
| [Update.cmd](../Update.cmd) | Prepare and verify the current upstream revision using the same installer. |
| [Repair-Shortcut.cmd](../Repair-Shortcut.cmd) | Recreate the desktop shortcut for this installation's current path. |

The default recipe is [projects/deepseek.json](../projects/deepseek.json), pointing to `deepseek-ai/deepseek-harness` and its `master` branch. The default installation folder is `installed/deepseek` beside these scripts. Downloading a newer revision does not require merging a fork or recreating wrapper source.

The [Manage.ps1](../Manage.ps1) `Plan` action displays the selected recipe and destination without installing anything. Its `InstallRoot` parameter selects another destination, `Ref` selects an upstream branch/tag/commit, and `NoShortcut` suppresses shortcut creation. The command files forward these parameters to the script.
