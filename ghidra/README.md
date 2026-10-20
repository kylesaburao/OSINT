# Ghidra + GhidraMCP for Claude Code (macOS)

How to set up [Ghidra](https://github.com/NationalSecurityAgency/ghidra) and [GhidraMCP](https://github.com/LaurieWired/GhidraMCP) on a Mac so Claude Code can decompile, browse and annotate the program you have open in Ghidra.

```
Claude Code ──stdio──▶ ghidra-mcp (run by uvx) ──HTTP 127.0.0.1:8080──▶ GhidraMCP plugin inside Ghidra
```

The MCP server is only a bridge. It works only while Ghidra is running, the plugin is enabled, and a program is open in the CodeBrowser.

## Prerequisites

- macOS with [Homebrew](https://brew.sh).
- Claude Code.

Everything else is installed by the steps below.

## 1. Install the tools

```sh
brew install ghidra                  # Ghidra, plus openjdk@21 as a dependency
brew install uv                      # provides uvx, which runs the MCP server
brew pin ghidra                      # keep `brew upgrade` from moving Ghidra past the GhidraMCP version
```

Homebrew installs Ghidra as a formula, not an app bundle. You launch it from the terminal with `ghidraRun`, and nothing appears in `/Applications`. The installation lives at `$(brew --prefix ghidra)/libexec`.

**Java:** Ghidra needs JDK 21 or newer. The Homebrew `ghidraRun` wrapper uses Homebrew's `openjdk@21`, but a `JAVA_HOME` already set in your shell takes precedence. If Ghidra fails to start, check `echo $JAVA_HOME` and `java -version`. Run `unset JAVA_HOME` before `ghidraRun` to use Homebrew's JDK.

## 2. Check that the versions match

Each GhidraMCP release is built for one exact Ghidra version, and Ghidra refuses extensions built for a different version.

1. Get your Ghidra version:
   ```sh
   brew info ghidra | head -1        # e.g. "==> ghidra: stable X.Y.Z (bottled)"
   ```
2. Open <https://github.com/LaurieWired/GhidraMCP/releases/latest> and download `GhidraMCP-release-*.zip`.
3. Unzip it. Inside is an extension zip named `ghidra_<ghidra-version>_PUBLIC_<date>_GhidraMCP.zip`. The `<ghidra-version>` in its name must equal the version from step 1.

As of 2026-10-19, Homebrew and GhidraMCP release 2.0 both target Ghidra 12.1.4.

If the versions differ, wait for a GhidraMCP release that matches. Keep Ghidra pinned (step 1) until then.

## 3. Install the GhidraMCP extension in Ghidra

1. Start Ghidra:
   ```sh
   ghidraRun
   ```
   On first launch, accept the user agreement.
2. In the Project window, choose **File → Install Extensions**, click **+**, and select the **inner** `ghidra_…_GhidraMCP.zip` from step 2, not the outer `GhidraMCP-release-*.zip`.
3. Quit and restart Ghidra.
4. Create a project with **File → New Project**. Import a binary with **File → Import File**, then double-click it to open the CodeBrowser and let auto-analysis run.
5. In the CodeBrowser, choose **File → Configure → Developer** and make sure **GhidraMCPPlugin** is ticked.

The plugin serves HTTP on `http://127.0.0.1:8080/`. To use another port, go to **Edit → Tool Options → GhidraMCP HTTP Server** in the CodeBrowser, and use the same port in step 4.

Check that the plugin responds. With Ghidra still open, run:

```sh
curl http://127.0.0.1:8080/health
curl http://127.0.0.1:8080/program_info
```

## 4. Register the MCP server with Claude Code

```sh
claude mcp add -s user ghidra -- uvx --from git+https://github.com/LaurieWired/GhidraMCP ghidra-mcp --ghidra-server http://127.0.0.1:8080/
```

- `-s user` makes the server available in every project. Leave it out to register it for the current directory only.
- `uvx` downloads and runs `ghidra-mcp` straight from the GitHub repo, so no manual Python setup is needed.
- If you changed the plugin port, change `--ghidra-server` to match.

Check the registration:

```sh
claude mcp list          # expect: ghidra: uvx --from ... - ✔ Connected
```

## 5. Use it

1. Run `ghidraRun` and open a program in the CodeBrowser.
2. Start `claude` in any directory.
3. Ask, for example: *"Call get_program_info, then list the functions and decompile main."*

The available tools include `get_program_info`, `list_functions`, `decompile_function`, `get_xrefs_to`, `list_strings` and `rename_function`. Run `/mcp` inside Claude Code to see them all.

## Other MCP clients

Clients that take a JSON config use the same command:

```json
{
  "mcpServers": {
    "ghidra": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/LaurieWired/GhidraMCP", "ghidra-mcp", "--ghidra-server", "http://127.0.0.1:8080/"]
    }
  }
}
```

Clients that connect by URL need the server started separately:

```sh
uvx --from git+https://github.com/LaurieWired/GhidraMCP ghidra-mcp --ghidra-server http://127.0.0.1:8080/ --transport streamable-http --mcp-host 127.0.0.1 --mcp-port 8081
```

Then add `http://127.0.0.1:8081/mcp` as a remote server.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `curl` to `127.0.0.1:8080` fails with "connection refused" | Ghidra isn't running, no program is open in the CodeBrowser, GhidraMCPPlugin isn't enabled (**File → Configure → Developer**), or the port differs from the one in Tool Options. |
| Install Extensions rejects the zip, or the plugin is missing after restart | Ghidra and extension versions differ (see step 2), or you selected the outer release zip instead of the inner `ghidra_…_GhidraMCP.zip`. |
| `ghidraRun` fails with a Java error | An old JDK in `JAVA_HOME` is taking precedence. See the Java note in step 1. |
| `claude mcp list` shows the server failing | Make sure `uvx --version` works. Run the `uvx …` command from step 4 by hand to see its error. |
| Claude's Ghidra tools return errors or empty results | Ghidra is closed or showing a different program. Reopen the binary in the CodeBrowser. |

## Update or uninstall

- **Update Ghidra:** only once a matching GhidraMCP release exists, run `brew unpin ghidra && brew upgrade ghidra && brew pin ghidra`. Then install the matching extension zip (step 3).
- **Update the MCP server:** `uvx` caches the package. Run `uvx --refresh --from git+https://github.com/LaurieWired/GhidraMCP ghidra-mcp --help` once to re-fetch it from GitHub.
- **Remove the extension:** in Ghidra, go to **File → Install Extensions**, untick GhidraMCP, and restart.
- **Uninstall everything:**
  ```sh
  claude mcp remove -s user ghidra
  brew unpin ghidra
  brew uninstall ghidra
  ```
