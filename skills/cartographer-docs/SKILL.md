---
name: cartographer-docs
description: >
  Deep dive into codebase architecture with text explanations.
  Use when the user says: "explain the architecture", "how does X work",
  "refresh my memory", "deep dive into", "explain this system",
  "I forgot how X works", "walk me through".
allowed-tools: Read, Glob, Grep
metadata:
  author: stevederico
  version: "1.0.0"
  argument-hint: <subsystem> or "question"
---

# cartographer-docs

You provide text-based architecture explanations and deep dives. Companion to **cartographer** (visual HTML diagrams): you explain *how* things work and *why* they're built that way. Output is in-conversation markdown with file:line references.

---

## Workflow

### Step 1: Determine Scope

Parse the user's request to determine what they need:

| Input | Action |
|-------|--------|
| `/cartographer-docs` (no args) | Full system overview |
| `/cartographer-docs <subsystem>` | Deep dive into that subsystem |
| `/cartographer-docs "question"` | Answer the specific question |

**Known subsystems** (read from project's `docs/ARCHITECTURE.md` if it exists):
- `audio` — Recording, transcription, TTS pipeline
- `chat` — Message flow, streaming, agent loop
- `tools` — Tool system, permissions, execution
- `servers` — The 4 servers and why they exist
- `swift` — Swift app structure and key components
- `data` — Data storage, SQLite, file locations

### Step 2: Gather Context

1. **Check for existing docs:**
   - Read `docs/ARCHITECTURE.md` if it exists (canonical reference)
   - Read `docs/DECISIONS.md` if it exists (the "why" behind decisions)
   - Read `AGENTS.md` and/or `CLAUDE.md` for project-specific conventions (often one symlinks to the other)

2. **For subsystem deep dives, read the relevant source files:**
   - Use Glob to find related files
   - Use Grep to trace function calls and data flow
   - Read key files to understand implementation

3. **Build a mental model:**
   - Identify entry points
   - Trace data flow through the system
   - Note key classes/functions and their responsibilities
   - Identify cross-cutting concerns (auth, logging, errors)

### Step 3: Generate Explanation

Structure your response based on scope:

**Full system overview:**
```markdown
## The Big Picture
[1-2 paragraph summary of what the system does]

## Architecture Diagram (ASCII)
[Simple box diagram showing major components]

## Key Components
[Table of components with one-line descriptions]

## Data Flow
[How a typical request flows through the system]

## Quick Reference
[Ports, key files, common commands]
```

**Subsystem deep dive:**
```markdown
## [Subsystem Name] — What It Does
[1 paragraph summary]

## How It Works
[Step-by-step walkthrough with file:line references]

## Key Files
[Table: file path | responsibility]

## Data Flow
[Trace a request/action through the subsystem]

## Why It's Built This Way
[Architectural decisions and tradeoffs]

## Common Tasks
[How to modify/debug this subsystem]
```

**Specific question:**
```markdown
## [Restate question as heading]

[Direct answer with file:line references]

[Code snippets if helpful]

[Related context they might need]
```

### Step 4: Include References

Always include concrete references:
- `file.swift:123` — line numbers for key code
- Link related subsystems: "See also: `/cartographer-docs chat`"
- Mention relevant decisions from `docs/DECISIONS.md`

### Step 5: Offer Follow-ups

End with suggested next steps:
```markdown
---
**Related deep dives:**
- `/cartographer-docs <related-subsystem>`
- `/cartographer` for visual representation
```

---

## Output Guidelines

1. **Be concrete** — Always reference actual files and line numbers
2. **Trace data flows** — Show how data moves through the system
3. **Explain the "why"** — Don't just describe, explain decisions
4. **Use ASCII diagrams** — Simple boxes and arrows for flows
5. **Keep it scannable** — Use tables, headers, bullet points
6. **Stay current** — Read actual code, don't rely on outdated docs

---

## Example Outputs

### `/cartographer-docs` (full overview)
```markdown
## The Big Picture

Dottie is a native macOS voice assistant that runs AI locally...

## Architecture

┌──────────────┐     ┌─────────────────────────┐
│  Dottie.app  │────▶│  Gateway (1317)         │
│   (SwiftUI)  │     │  ┌───────┐  ┌───────┐   │
└──────────────┘     │  │ Audio │  │ LLM   │   │
                     │  │ 1315  │  │ 1316  │   │
                     │  └───────┘  └───────┘   │
                     └─────────────────────────┘

## Key Components
| Component | Port | Purpose |
|-----------|------|---------|
| Gateway   | 1317 | SSE streaming, tools, supervision |
...
```

### `/cartographer-docs audio`
```markdown
## Audio Pipeline — What It Does

Handles voice recording, speech-to-text transcription, and text-to-speech playback...

## How It Works

1. **Recording starts** (`AudioManager.swift:77`)
   - AVAudioEngine captures audio
   - Writes to ~/.dottie/recordings/

2. **Transcription** (`AudioManager.swift:220`)
   - Sends WAV to Audio Server (port 1315)
   - MLX STT model processes audio
...
```

### `/cartographer-docs "why does the gateway supervise servers?"`
```markdown
## Why Does the Gateway Supervise Servers?

The gateway (port 1317) manages the Audio (1315), LLM (1316), and Store (1318)
servers instead of Swift because:

1. **Node.js is better at process management** (`supervisor.js:287-406`)
   - Clean child_process spawning
   - Proper signal handling (SIGTERM → SIGKILL)
   - Health polling with exponential backoff

2. **Swift process management is clunky**
   - NSTask/Process API is verbose
   - No built-in health monitoring
...
```

---

## Maintaining Architecture Docs

When you make significant changes to the codebase:

1. Update `docs/ARCHITECTURE.md` with structural changes
2. Update `docs/DECISIONS.md` with new architectural decisions
3. These become the canonical reference for future `/cartographer-docs` calls
