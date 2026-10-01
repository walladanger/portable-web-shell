# Portable Web Shell

Keep application code and the reusable shell separate. Project recipes select supported upstream applications; do not patch downloaded application UI or runtime sources to integrate them with the shell.

The current installer supports public GitHub source ZIPs and a root package.json declaring an exact pnpm version. Only DeepSeek is a verified application recipe. Do not imply that changing a repository name makes an arbitrary project compatible; different runtimes or readiness protocols need an explicit adapter.

Keep the shell's local controls separate from the application WebView. Keep all owned runtime profiles, logs and data under the installation root. Preserve contained process ownership and clean up the owned backend tree on window exit.

Relevant checks are `cargo test --locked` in `shell/`, `Test-Wrapper.ps1`, and `Test-Install.ps1`; use `Test-Window.ps1` on an interactive Windows desktop for native control changes. Build with `Build-Shell.ps1` and package with `Package.ps1`. Inspect the curated package for installed applications or credentials before publishing. README.md and README.zh.md describe the same behavior.
