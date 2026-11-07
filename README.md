# rpi-skills
Adaptation of the Research, Plan, Implement recipe pattern from Goose AI implemented as standard skills.

## Layout

- `skills/`: The RPI skills (`rpi-research`, `rpi-plan`, `rpi-iterate`, `rpi-implement`, and the research sub-skills `rpi-codebase-locator`, `rpi-codebase-analyzer`, and `rpi-pattern-finder`).
- `plugins/rpi-ctags/`: A plugin that provides Ctags symbol tools (`search_symbols`, `list_symbols`, `generate_ctags`) as an MCP server. These are the same tools [rpi-agent](https://github.com/dcode/rpi-agent) has built in. The research skills use them when they're available and fall back to text search when they aren't.
- `.claude-plugin/marketplace.json`: Claude Code plugin marketplace that publishes both of the above.

## Installation

### Claude Code

```shell
claude plugin marketplace add dcode/rpi-skills
claude plugin install rpi@rpi-skills        # the skills
claude plugin install rpi-ctags@rpi-skills  # the ctags MCP tools
```

`rpi-ctags` requires [uv](https://docs.astral.sh/uv/) and [Universal Ctags](https://ctags.io/) (Exuberant Ctags also works; BSD/POSIX `ctags` does not). See [plugins/rpi-ctags/README.md](plugins/rpi-ctags/README.md).

### rpi-agent or other agents

Extract the skills and the tools into your agents directory. This creates `${HOME}/.agents/skills/` and `${HOME}/.agents/plugins/rpi-ctags/`:

```shell
mkdir -p ${HOME}/.agents
cd ${HOME}/.agents
curl -L 'https://github.com/dcode/rpi-skills/archive/refs/heads/main.tar.gz' | tar --strip-components 1 -xzvf - rpi-skills-main/skills rpi-skills-main/plugins
```

rpi-agent has the ctags tools built in, so it only needs `skills/`. Other MCP-capable agents can run the tools as a stdio server with `uv run --script ${HOME}/.agents/plugins/rpi-ctags/server/ctags_server.py`, started from the repository being analyzed.
