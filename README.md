# Pi Minimal Subagents

Minimal named subagent tool for Pi.

## Installation

Install directly from GitHub with Pi:

```bash
pi install git:github.com/elpapi42/pi-minimal-subagent
```

Then restart Pi, or run `/reload` in an existing session if your Pi version supports extension reloads.

For local development from this checkout:

```bash
cd /home/whitman/minimal-subagent/pi-minimal-subagents
pi -e .
```

## Usage

It registers one tool:

```json
{ "agent": "scout", "task": "Inspect the auth flow and report risks." }
```

There are no built-in parallel, chain, pool, or orchestrator modes. If the parent agent wants parallel subagents, it should call `subagent` multiple times in the same turn and let Pi execute those tool calls concurrently.

## Agent files

Agents are Markdown files with YAML frontmatter:

```markdown
---
name: scout
description: Fast codebase reconnaissance
model: claude-haiku-4-5
extensions: npm:some-pi-extension
---
You are a fast codebase scout. Return dense findings for the parent agent.
```

Loaded from:

- Pi's global agent directory, usually `~/.pi/agent/agents/*.md` and honoring `PI_CODING_AGENT_DIR`
- `.pi/agents/*.md` in the current project or an ancestor directory

Project agents override user agents with the same name.

Supported optional frontmatter: `model`, `extensions`, `skills`, and `thinking`.

Subagents use Pi's default enabled tools. This extension does not read `tools` frontmatter and does not pass `--tools` to child Pi processes. Extra tools should come from configured extensions.

## Settings

Global settings live in Pi's agent settings file (usually `~/.pi/agent/settings.json`; honors `PI_CODING_AGENT_DIR`). Project settings live in `.pi/settings.json` and override global settings.

```jsonc
{
  "pi-minimal-subagent": {
    "model": null,
    "extensions": [
      "git:git@github.com:elpapi42/pi-codemapper.git",
      "npm:pi-rtk-optimizer"
    ]
  }
}
```

`model` is the default model for spawned subagents. Agent frontmatter `model` overrides it.

`extensions` is tri-state, matching `pi-fork`:

- `null` or omitted: child subagents load normal Pi extensions from settings and auto-discovery.
- `[]`: child subagents run with `--no-extensions` and no default extra extensions.
- non-empty array: child subagents run with `--no-extensions`, then explicitly load those extensions.

Agent frontmatter `extensions` are always appended as explicit `--extension` entries. With `extensions: null`, they are added on top of normal Pi extension loading; with `[]` or a non-empty array, they are the only additions besides the configured list.

The extension does not block recursive usage. If a user loads this extension inside a subagent, nested subagent calls are allowed.

## Development

From this directory:

```bash
npm run typecheck
pi -e .
```
