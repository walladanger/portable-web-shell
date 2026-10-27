## Update and move

Updates download each source revision into `app/versions/<commit>`, build it, and test its authenticated Web entry using a separate verification home. `launch.json` is replaced only after that test succeeds. Earlier revisions remain in the folder; they are not automatically deleted or selected as a downgrade. DeepSeek's persisted data has its own compatibility rules, so retaining an older build does not establish that it can read newer sessions.

Close the application before updating. An installation lock rejects simultaneous installers. The recorded upstream commit and runtime versions are in `installed.json`.

Close the application and move the installer bundle and its `installed` folder together. After changing the installation path or machine, run `Install.cmd` again: pnpm dependencies can contain Windows junctions that need relinking, and the shortcut contains an absolute destination. The installer retains application data, moves the previous generated dependency directory into `data/dependency-backups` when the recorded root changed, rebuilds dependencies and verifies the app, and recreates the shortcut. Shortcut repair alone does not reinstall dependencies. Paths to workspaces outside the installation may need to be selected again on another computer.
