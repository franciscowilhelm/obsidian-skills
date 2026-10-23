---
name: obsidian-cli
description: Interact with Obsidian vaults using the Obsidian CLI to read, create, search, and manage notes, tasks, properties, bases, and more. Use when the user asks to interact with their Obsidian vault, manage notes, search vault content, or perform vault operations from the command line.
---

# Obsidian CLI

Use the `obsidian` CLI to interact with a running Obsidian instance. Requires Obsidian to be open.

## Sandbox rules

Bash runs in a sandbox. Only a plain `obsidian ...` call is let out of it to reach the running app, and every call waits for the user's approval.

- Call `obsidian` as one plain command. No pipes, redirects, `2>&1`, `cd ... &&` or other chaining, `$(...)`, full binary path, or calls from scripts.
- Narrow output with the CLI's own options (`limit=`, `total`, `format=`, `path=`, `folder=`, `file=`) instead of `| head` or `grep`. If more filtering is needed, run the plain command first and process its output in a separate step. No command writes its output to a file, so don't add a shell redirect.
- Each call is an approval prompt. Use as few calls as sensible (for example `search:context` instead of `search` followed by several `read`s), and say what each call will do before running it.
- "The CLI is unable to find Obsidian" after a non-plain call means the sandbox caught it. Rerun it plain. Don't suggest settings changes to get around this.
- `eval` and all `dev:*` commands are blocked by deny rules. Don't use them, or `devtools`.
- Ask before running commands that change the app rather than notes: `plugin:install`, `plugin:enable`, `plugins:restrict off`, `theme:install`, `command id=...`, `templater:create-from-template` (Templater can run scripts), `restart`. Never use `delete ... permanent` unless the user asks for it.

## Command reference

Run `obsidian help` to see all available commands, or `obsidian help <command>` for one. This is always up to date. Full docs: https://help.obsidian.md/cli

## Syntax

**Parameters** take a value with `=`. Quote values with spaces:

```bash
obsidian create name="My Note" content="Hello world"
```

**Flags** are boolean switches with no value:

```bash
obsidian create name="My Note" overwrite open
```

For multiline content use `\n` for newline and `\t` for tab.

## File targeting

Many commands accept `file` or `path` to target a file. Without either, the active file is used.

- `file=<name>` — resolves like a wikilink (name only, no path or extension needed)
- `path=<path>` — exact path from vault root, e.g. `folder/note.md`

## Vault targeting

Commands target the most recently focused vault by default. Use `vault=<name>` as the first parameter to target a specific vault. The vault in this setup is `Main`:

```bash
obsidian vault=Main search query="test"
```

## Common patterns

```bash
obsidian read file="My Note"
obsidian create name="New Note" content="# Hello" template="Template"
obsidian append file="My Note" content="New line"
obsidian search query="search term" limit=10
obsidian search:context query="search term" path="Projects" limit=5
obsidian property:set name="status" value="done" file="My Note"
obsidian tasks daily todo
obsidian tags sort=count counts
obsidian backlinks file="My Note"
obsidian base:query file="Projects" view="Active" format=md
```

Files are not opened unless you pass `open`. Use `total` on list commands to get a count, and `format=json` where offered for output that is easier to process.
