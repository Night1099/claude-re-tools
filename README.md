# vibe-re

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin for [Vibe-Reverse-Engineering](https://github.com/Ekozmaster/Vibe-Reverse-Engineering). Built on top of the retools, livetools, dx9 tracer, and rtx_remix_tools provided by that project.

Provides agents, commands, and skills for analyzing game binaries, porting DX9 games to RTX Remix, and running interactive RE sessions with parallel static + dynamic analysis.

## Install

1. Clone the tooling repo and install dependencies:

```bash
git clone https://github.com/Ekozmaster/Vibe-Reverse-Engineering.git
cd Vibe-Reverse-Engineering
pip install -r requirements.txt
```

2. Install the plugin in Claude Code:

```
/plugin marketplace add Night1099/claude-re-tools
/plugin install vibe-re@claude-re-tools
/reload-plugins
```

### Installing via the Claude VSCode Extension

1. Run this in PowerShell:

```powershell
git config --global url."https://github.com/".insteadOf "git@github.com:"
```

2. Open Claude extension, type the `/plugin` command and click **Manage Plugins**
3. Go to the **Marketplace** tab and type `Night1099/claude-re-tools`
4. Go back to the **Plugins** tab and click **Install** on `vibe-re@claude-re-tools`

## What's included

### Agents

| Agent | Description |
|-------|-------------|
| **dx9-analyzer** | Analyzes DX9 frame trace JSONL captures — render passes, shader maps, constant provenance, pipeline diagrams |
| **ffp-porter** | Ports DX9 shader-based games to fixed-function pipeline for RTX Remix compatibility |
| **re-investigate** | Static-only RE analysis — string search, decompilation, callgraphs, struct reconstruction |

### Commands

| Command | Description |
|---------|-------------|
| `/re <path_to_exe> <what to investigate>` | Start an RE session — parallel static + dynamic analysis on a game binary |
| `/kb-update <project> <entry>` | Append a discovery (function, global, struct) to a project's `kb.h` knowledge base |
| `/new-project <name>` | Scaffold a new RE project under `patches/` with kb.h template and traces directory |

### Skills

| Skill | Description |
|-------|-------------|
| **re-session** | Full interactive RE session — spawns background static analysis while you run the game for live Frida-based dynamic tracing. Combines both tracks to find, trace, and patch behavior. |

## Dependencies

This plugin expects the following tools to be available in the working directory:

- **retools** — static analysis toolkit (`retools/search.py`, `retools/decompiler.py`, `retools/xrefs.py`, etc.)
- **livetools** — Frida-based dynamic analysis (`python -m livetools attach/trace/bp/mem`)
- **graphics.directx.dx9.tracer** — DX9 frame capture and analysis
- **rtx_remix_tools** — FFP proxy template for RTX Remix porting

## Project structure

RE projects live under `patches/<project>/` with:
- `kb.h` — knowledge base of discovered functions, globals, and structs
- `traces/` — JSONL trace data from livetools and the DX9 tracer
- `findings.txt` — progressive findings written by background agents

## Troubleshooting

**"Host key verification failed" when installing the plugin:**

Run this in your terminal to force git to use HTTPS instead of SSH:

```bash
git config --global url."https://github.com/".insteadOf "git@github.com:"
```

Then retry the `/plugin install` command.

## License

MIT
