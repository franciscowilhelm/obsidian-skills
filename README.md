Agent Skills for Obsidian, adapted for a sandboxed Claude Code setup.

This is a fork of [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills). It keeps a reduced set of skills and changes them to work in a Claude Code setup where every shell command runs in the sandbox and only plain `obsidian` calls, each approved by the user, can reach the running Obsidian app. Use the upstream repo if you don't run Claude Code this way.

## Skills

| Skill                                         | Description                                                                                                                                                                                |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [obsidian-markdown](skills/obsidian-markdown) | Create and edit [Obsidian Flavored Markdown](https://help.obsidian.md/obsidian-flavored-markdown) (`.md`) with wikilinks, embeds, callouts, properties, and other Obsidian-specific syntax |
| [obsidian-bases](skills/obsidian-bases)       | Create and edit [Obsidian Bases](https://help.obsidian.md/bases/syntax) (`.base`) with views, filters, formulas, and summaries                                                             |
| [obsidian-cli](skills/obsidian-cli)           | Read, search and manage notes in a running vault via the [Obsidian CLI](https://help.obsidian.md/cli), following the sandbox rules below                                                   |

### Changes from upstream

- Removed `defuddle`, `knap` and `json-canvas`. Defuddle fetches web pages from the shell, which the sandbox network proxy blocks for most hosts, and both Defuddle and Knap need a global `npm install`, which the sandbox can't write. Canvas isn't used.
- `obsidian-cli` tells Claude to call the CLI only as a single plain command, and drops the plugin development commands (`eval`, `dev:*`, `devtools`) that the setup blocks.
- `obsidian-markdown` and `obsidian-bases` verify their result with a plain CLI call (`read`, `links`, `base:query`) instead of reading vault files directly. Otherwise they are unchanged.
- The plugin and marketplace are both named `obsidian-sandboxed` so they can be installed alongside the upstream `obsidian@obsidian-skills` without a name clash.

## Required Claude Code settings

The skills assume the settings in [`settings.example.json`](settings.example.json). Merge them into `~/.claude/settings.json`. Claude can't do this for you, because the sandbox write-protects that file.

| Setting | Effect |
| --- | --- |
| `sandbox.enabled`, `failIfUnavailable` | Every Bash command runs in the macOS Seatbelt sandbox, and Claude Code refuses to start without it. |
| `sandbox.allowUnsandboxedCommands: false` | There's no "retry outside the sandbox" fallback. |
| `sandbox.excludedCommands: ["obsidian *"]` | The only exception: plain `obsidian ...` calls run outside the sandbox, so they can reach the running app through its socket (`~/.obsidian-cli.sock`). |
| `permissions.ask: ["Bash(obsidian *)"]` | Every CLI call waits for your approval. |
| `permissions.deny` on `eval` and `dev*` | Blocks the commands that run JavaScript inside Obsidian. They have full access to the machine, so they would be a way out of the sandbox. `dev*` covers all `dev:` commands and `devtools`. Both the `obsidian eval` and `obsidian vault=... eval` forms are covered. |
| `extraKnownMarketplaces`, `enabledPlugins` | Install the skills from this fork. |

Add your own filesystem, network and credential rules under `sandbox` as needed. The skills don't depend on them.

### Why CLI calls must be plain

A call leaves the sandbox only when its whole text matches `obsidian *`. Pipes (`| head`), redirects (`> file`, `2>&1`), chains (`cd ... &&`), `$(...)`, the full binary path, and calls from scripts or `make` targets all stay in the sandbox. There the CLI fails with "The CLI is unable to find Obsidian".

This is intended. The only way out of the sandbox is a visible `obsidian` call that you approve. Don't widen it with `sandbox.network.allowUnixSockets` for the socket, `allowAppleEvents`, or broader `excludedCommands`. Each of these would let any sandboxed process, scripts included, drive Obsidian without approval.

## Installation

```
/plugin marketplace add franciscowilhelm/obsidian-skills
/plugin install obsidian-sandboxed@obsidian-sandboxed
```

You can also add the `extraKnownMarketplaces` and `enabledPlugins` entries from [`settings.example.json`](settings.example.json) yourself. If you used the upstream plugin before, disable `obsidian@obsidian-skills` so the skills don't load twice.

The skills follow the [Agent Skills specification](https://agentskills.io/specification), so other agents can read them too, but the sandbox rules only apply to Claude Code.

## Syncing with upstream

The changes are kept small so upstream updates can be merged:

```sh
git remote add upstream https://github.com/kepano/obsidian-skills.git
git fetch upstream
git merge upstream/main
```

If upstream edits one of the removed skills, git reports a modify/delete conflict. Resolve it with `git rm` on that file.
