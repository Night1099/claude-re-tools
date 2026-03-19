---
name: kb-update
description: Append a discovery to the project's knowledge base (kb.h)
argument-hint: "<project> <entry>"
allowed-tools: ["Read", "Edit", "Write", "Glob"]
---

Append a reverse engineering finding to a project's knowledge base file.

## Input

The user provides:
1. A project name (subdirectory under `patches/`)
2. A kb.h entry — one of:
   - Function signature: `@ 0xADDR returntype calling_convention FuncName(args)`
   - Global variable: `$ 0xADDR Type g_varName`
   - Struct/enum definition
   - Free-form note as a C comment

If the user omits the project name, check which `patches/*/kb.h` files exist and ask which project to update. If only one exists, use it.

## Process

1. Read the current `patches/<project>/kb.h` file (create it if it doesn't exist, with a header comment `// Knowledge base for <project>`)
2. Determine where the new entry belongs:
   - Struct/enum definitions go near the top, grouped with other type definitions
   - `@` function signatures go in a function signatures section, sorted by address
   - `$` globals go in a globals section, sorted by address
   - Comments go near related entries
3. Check for duplicates — if the same address already has an entry, update it rather than adding a duplicate
4. Write the updated file

## Format Rules

- Addresses are hex with `0x` prefix
- Function signatures include calling convention (`__cdecl`, `__thiscall`, `__stdcall`, `__fastcall`)
- Struct fields include byte offset comments: `float x; // +0x00`
- One blank line between sections (types, functions, globals)

$ARGUMENTS
