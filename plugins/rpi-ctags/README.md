# rpi-ctags

MCP server that exposes Ctags symbol tools to agents. The tool names, arguments, and output format are the same as the built-in tools in [rpi-agent](https://github.com/dcode/rpi-agent), so the RPI skills work the same way under either agent.

| Tool | Read-only | Purpose |
|---|---|---|
| `search_symbols` | yes | Regex search over symbol names, with optional filters for path, kind, and language. Can include source context. |
| `list_symbols` | yes | Lists every indexed declaration under a file or directory, sorted by `file:line`. |
| `generate_ctags` | no | Writes a `tags` index at the repo root if none exists. Pass `force=True` to rebuild it. |

In Claude Code the tools appear as `mcp__plugin_rpi-ctags_ctags__<tool>`.

## Requirements

- [uv](https://docs.astral.sh/uv/): runs `server/ctags_server.py` as a PEP 723 script and installs its dependency (`mcp`) on first launch.
- [Universal Ctags](https://ctags.io/) (preferred) or Exuberant Ctags on `PATH`. BSD/POSIX `ctags` is rejected. On macOS, run `brew install universal-ctags`.

## Behavior

- **Repository root**: the first of `RPI_REPO_ROOT`, `CLAUDE_PROJECT_DIR`, or the working directory, widened to its Git top-level. All paths are confined to this root.
- **Index location**: the server reads the first of `tags`, `.tags`, or `.git/tags` that exists, and `generate_ctags` writes `tags`. rpi-agent uses the same rules, so both share one index. Consider adding `tags` to your `.gitignore`.
- **Staleness**: the index is not rebuilt automatically. Run `generate_ctags(force=True)` after large changes.

## Keeping in sync with rpi-agent

The Ctags logic is a copy of `src/rpi_agent/tools/ctags.py` from rpi-agent, so this server doesn't need rpi-agent's heavier dependencies. When that file changes, port the changes here.

## Running standalone

```shell
cd /path/to/repo
uv run --script /path/to/plugins/rpi-ctags/server/ctags_server.py
```

This works with any MCP client that supports stdio servers.
