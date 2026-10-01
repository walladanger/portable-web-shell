# Source and attribution

Portable Web Shell was extracted from `portable-desktop/` in [walladanger/deepseek-harness at 489f579487e2e7e485c75ace850eca5eef9ac920](https://github.com/walladanger/deepseek-harness/tree/489f579487e2e7e485c75ace850eca5eef9ac920/portable-desktop). The standalone repository retains the MIT license, source assets, pinned Cargo dependencies, and third-party notices. The initial extraction corrects the successful installer test's exit code when invoked by GitHub Actions.

The DeepSeek application is a separate upstream download selected by `projects/deepseek.json`; it is not vendored in this repository or included in the compiled wrapper bundle. Installed applications, credentials, sessions, logs, caches, and build output are excluded from Git and release packaging.
