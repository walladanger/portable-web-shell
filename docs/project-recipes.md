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
