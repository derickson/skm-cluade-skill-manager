# skm CLI Reference

`skm` is a small Python CLI for bootstrapping the SKM skill into Claude Code projects. It requires no dependencies beyond the Python standard library.

## Installation

```bash
cd /path/to/claude-skill-manager
make install      # creates ~/.local/bin/skm -> skill_manager.py
make uninstall    # removes ~/.local/bin/skm
```

Ensure `~/.local/bin` is on your `PATH`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Verify:

```bash
skm --help
```

## How it works

`skm` derives the library location from its own path at runtime:

```python
REPO_DIR = Path(__file__).resolve().parent
```

This means the CLI works correctly regardless of where the repo is cloned. If you move the repo, re-run `skm install` in your projects to update the config.

---

## Commands

### `skm install [target_project_path]`

Bootstraps SKM into a Claude Code project. Does two things:

**1. Writes `~/.config/skm/config.json`**

```json
{
  "repo_path": "/path/to/claude-skill-manager"
}
```

This file is the bridge between the CLI and the in-Claude SKM skill. It tells the skill where the library lives. Running `skm install` in any project always refreshes this config to point at the current repo location.

**2. Copies `src/skills/skm/` into the target project**

Source: `<repo>/src/skills/skm/`
Destination: `<target>/.claude/skills/skm/`

If the destination already exists, you are prompted:

```
'skm' already exists in /path/to/project. Overwrite? [y/N]
```

Enter `y` to overwrite, anything else to skip the copy (config is still written).

**Arguments:**

| Argument | Default | Description |
|---|---|---|
| `target_project_path` | `.` (current directory) | Path to the Claude Code project to install SKM into |

**Examples:**

```bash
# Install into the current directory
skm install

# Install into a specific project
skm install ~/dev/my-project

# Install into multiple projects
skm install ~/dev/project-a
skm install ~/dev/project-b
```

---

## Config file

`~/.config/skm/config.json` is written by `skm install` and read by the in-Claude SKM skill at runtime.

```
~/.config/skm/config.json
```

```json
{
  "repo_path": "/Users/you/dev/claude-skill-manager"
}
```

`repo_path` is the absolute path to the SKM library repo. The in-Claude skill uses it to locate `.claude/commands/` and `.claude/skills/` when listing or copying library content.

If this file is missing or has a stale path, the in-Claude skill will tell you to re-run `skm install`.

---

## Scope

The CLI handles bootstrapping only. All library management (listing, installing, and saving skills and commands) is done interactively inside Claude Code via the SKM skill. See the main [README](../README.md) for the in-Claude workflow.
