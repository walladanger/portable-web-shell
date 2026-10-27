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
