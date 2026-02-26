---
name: noisily
description: Enable an audio beep notification when Claude finishes responding, for the current project directory.
---

# Enable beep notifications

Run the following commands to enable beep-on-response for this project:

```bash
mkdir -p "$HOME/.claude/beep_projects"
touch "$HOME/.claude/beep_projects/$(pwd | tr '/' '_')"
```

Then tell the user: "Beep notifications are now **on** for this directory. Use `/silent` to turn them off."

## Notes

- The flag file is stored in `~/.claude/beep_projects/` named after the current directory path (slashes replaced with underscores), e.g. `_mnt_d_Lab`.
- It is local to this project — other Claude Code windows on different projects are unaffected.
- Nothing is added to the project file tree or `.gitignore`.