# vibe-re

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin for autonomous reverse engineering. Provides agents, commands, and skills for analyzing game binaries, porting DX9 games to RTX Remix, and running interactive RE sessions with parallel static + dynamic analysis.

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
```

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

## License

MIT
