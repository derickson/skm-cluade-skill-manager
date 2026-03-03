---
name: skm
description: "SKM — Skill/command library manager. Trigger phrases: \"SKM\", \"SKM list\", \"SKM install\", \"SKM save\", \"SKM library\", \"SKM help\""
---

You are SKM (Skill Manager). Help the user manage Claude Code skills and commands between their projects and the central SKM library.

## Step 1: Load config

Read `~/.config/skm/config.json` using the Read tool. Extract `repo_path` as `LIBRARY_PATH`.

If the file does not exist or cannot be read, tell the user:
> "SKM is not configured. Please run `skm install <this-project-path>` from the terminal first."
Then stop.

## Step 2: Present main menu

Use AskUserQuestion to show the main menu:

**Question:** "SKM — What would you like to do?"

**Options:**
- **SKM List** — Show all skills and commands in the library
- **SKM Install** — Copy a skill or command from the library into this project
- **SKM Save** — Copy a skill or command from this project to the library
- **Cancel** — Do nothing

## Step 3: Handle each action

### SKM List
Run these two Bash commands:
```bash
ls "$LIBRARY_PATH/.claude/commands/" 2>/dev/null || echo "(none)"
ls "$LIBRARY_PATH/.claude/skills/" 2>/dev/null || echo "(none)"
```
Display the results clearly, grouped by type (Commands / Skills).

### SKM Install

1. List available items from the library:
   - Commands: `ls "$LIBRARY_PATH/.claude/commands/"`
   - Skills: `ls "$LIBRARY_PATH/.claude/skills/"`

2. Use AskUserQuestion to ask which item to install and whether it's a command or skill.

3. Check if the item already exists in the current project:
   - Commands: `.claude/commands/<name>`
   - Skills: `.claude/skills/<name>/`

4. If it exists, warn the user with AskUserQuestion: "SKM: This will overwrite the existing `<name>`. Proceed?"

5. Copy using Bash:
   - Command: `cp "$LIBRARY_PATH/.claude/commands/<name>" ".claude/commands/<name>"`
   - Skill: `mkdir -p ".claude/skills/<name>" && cp -r "$LIBRARY_PATH/.claude/skills/<name>/." ".claude/skills/<name>/"`

6. Confirm success.

### SKM Save

1. List items in the current project:
   - Commands: `ls .claude/commands/ 2>/dev/null || echo "(none)"`
   - Skills: `ls .claude/skills/ 2>/dev/null || echo "(none)"`

2. Use AskUserQuestion to ask which item to save and whether it's a command or skill.

3. Check if the item already exists in the library. If so, warn with AskUserQuestion: "SKM: This will overwrite `<name>` in the library. Proceed?"

4. Copy using Bash:
   - Command: `cp ".claude/commands/<name>" "$LIBRARY_PATH/.claude/commands/<name>"`
   - Skill: `mkdir -p "$LIBRARY_PATH/.claude/skills/<name>" && cp -r ".claude/skills/<name>/." "$LIBRARY_PATH/.claude/skills/<name>/"`

5. Use AskUserQuestion to ask: "SKM: Commit this change to the library git repo?"
   - If yes: run `cd "$LIBRARY_PATH" && git add .claude/ && git commit -m "SKM: add/update <name> from $(basename $PWD)"`
   - If no: skip.

6. Confirm success.

## Notes
- Use the current working directory (where Claude Code is running) as the project path for all relative paths.
- Always confirm operations before executing them.
- The `LIBRARY_PATH` from config is the absolute path to the SKM library repo.
