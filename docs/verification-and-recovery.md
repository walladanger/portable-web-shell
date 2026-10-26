## Verification and recovery

The installer uses the shell's `--smoke` mode to launch the real backend, exchange its token for a cookie, request the HTML Web UI, and stop the backend. Installation failures leave the previous active `launch.json` in place. This checks startup and the authenticated index, not every DeepSeek feature or API provider.

[Test-Wrapper.ps1](../Test-Wrapper.ps1) exercises the compiled shell using real HTTP and descendant Node processes. It covers fragmented stdout readiness, authenticated redirects, unauthorized responses, startup timeout, parent exit, descendant cleanup and launch-token log redaction. Rust tests cover readiness URL and owned-path validation.

[Test-Install.ps1](../Test-Install.ps1) checks long-path extraction, rejection of unsafe ZIP entries, diagnostic capture, and native exit codes under Windows PowerShell. [Test-Window.ps1](../Test-Window.ps1) checks original page rendering and native window controls on an interactive Windows desktop.

If installation or startup fails, inspect the newest file in `installed/deepseek/data/logs`. Installer scripts stop on nonzero command exit codes. A startup error also appears in the title strip; invalid launch files produce a native error dialog. Retry installation after correcting the reported problem. The `Verify` action runs the active configuration's HTTP check without opening the GUI.
