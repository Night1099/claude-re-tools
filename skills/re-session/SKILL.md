---
name: re-session
description: This skill should be used when the user provides a path to a game binary (.exe/.dll) and wants to investigate, find, trace, or patch behavior in it. A full file path implies the user has the game installed and can run it. Typical triggers include "find the culling in game.exe", "trace how rendering works", "I want to patch the resolution lock", "help me RE the save system in game.exe", "find and disable X in game.exe", or any request combining a binary path with behavioral investigation. Launches parallel static analysis in background while prompting the user to run the game for live Frida-based dynamic analysis. Falls back to static-only if the user cannot run the game.
---

# RE Session — Parallel Static + Dynamic Analysis

You (the parent) do TWO jobs: run livetools yourself AND coordinate with a background static agent. Stay active doing live analysis — never go idle waiting for the agent.

## YOUR ROLE vs AGENT'S ROLE

| YOU (parent) do DIRECTLY | AGENT (background) does |
|---|---|
| Talk to user | search.py strings/imports |
| livetools attach/trace/bp/mem | decompiler.py |
| Read findings file, trace leads live | xrefs.py, callgraph.py |
| Synthesize all findings | structrefs.py, rtti.py |

**You must NEVER run retools commands.** Only livetools. The agent handles all static analysis.

## Step 1: First Response — Ask User + Spawn Agent

Do BOTH in your first message:

**A) Text:** Tell user to launch the game (no mods/proxy, get to gameplay with 3D geometry, keep window focused).

**B) Spawn agent** using the Agent tool with `subagent_type: "vibe-re:re-investigate"` and `run_in_background: true`.

Keep the agent prompt SHORT — just the binary path and what to find:
```
Binary: <full_path_to_exe>
Find <what the user asked about>. Report function addresses, decompiled code, and kb.h entries.
```
Do NOT add search strategies, methodology, or step-by-step plans. The agent has its own process.

**If user can't run the game:** Spawn re-investigate in foreground (not background) for static-only analysis.

## Step 2: User Says Game Is Running — Attach and START WORKING

When user confirms, run these THREE things immediately:
```bash
python -m livetools attach <process_name>
python -m livetools modules
cat patches/<project>/findings.txt 2>/dev/null
```
The third command checks if the static agent already wrote findings.

Then **DO NOT STOP.** Keep running livetools commands. You are the dynamic analysis track.

**A) Explore live based on the investigation target:**
- Check the modules list for relevant DLLs (d3dx, physics, networking, etc.)
- Find exported functions of interest and trace calls to them
- Use `python -m livetools trace <addr> --count 20 --read "ecx; [esp+4]:4:uint32"` on candidate addresses

**B) If you don't have specific addresses yet**, do general exploration:
```bash
python -m livetools mem read <addr> 4 --as uint32
python -m livetools trace <addr> --count 30 --read "ecx; eax"
```

**C) Check the findings file after each livetools command completes.** The static agent writes results progressively to `patches/<project>/findings.txt`. Do NOT use TaskOutput — always read the actual file:
```bash
cat patches/<project>/findings.txt 2>/dev/null
```
Each time you read it, look for NEW addresses you haven't traced yet and trace them immediately.

## Step 3: Act on Static Agent Results

When the background agent completes (you get notified) or you find addresses in the findings file:
- Function addresses → trace them: `python -m livetools trace <addr> --count 20 --read "ecx; [esp+4]:4:uint32"`
- Global variables → read/toggle: `python -m livetools mem read <addr> 4 --as uint32` then `python -m livetools mem write <addr> "00000000"`
- Branch/comparison addresses → breakpoint: `python -m livetools bp add <addr>` → `watch` → `regs` → `resume`

**For patching:**
```bash
# NOP a conditional jump
python -m livetools mem write <addr> "90 90"
# Force a jump to always-taken
python -m livetools mem write <addr> "EB"
```

Tell user to check the game — "Does the behavior change?"

## Step 4: Report

- What the investigated system does (algorithm explanation)
- kb.h entries: `@ addr` functions, `$ addr` globals, struct definitions
- What was confirmed live vs static-only
- Patches applied or recommended

## User Interaction

- Tell user WHAT to do and WHY: "Walk around so the system is actively running"
- Give timing: "Play for ~30 seconds while I collect"
- Mention focus: "Keep game window focused"
- Confirm before writes: "I'll NOP the check at 0xADDR. Won't persist after restart. OK?"
- Tell them when done: "Alt-tab back, I have the data"
