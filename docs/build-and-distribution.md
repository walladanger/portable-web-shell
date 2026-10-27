## Build and distribution

The [Windows workflow](https://github.com/walladanger/portable-web-shell/actions/workflows/windows.yml) builds the reusable executable on Windows, runs Rust and executable behavior checks, and publishes a ZIP artifact containing the executable, scripts, recipes, licenses, and README. It does not install DeepSeek in CI or publish an app release automatically.

The bundle deliberately excludes installed applications, logs, credentials, dependency caches and Rust build output. The shell is built once for distribution; each user installation then downloads its upstream app. Update the shell separately when its own implementation changes.

[Package.ps1](../Package.ps1) creates that curated ZIP locally. The wrapper executable is unsigned; the workflow does not use a signing certificate. The wrapper is a separate artifact from DeepSeek's signed official desktop client.
