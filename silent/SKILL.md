---
name: silent
description: Disable the audio beep notification when Claude finishes responding, for the current project directory.
---

# Disable beep notifications

Run the following command to disable beep-on-response for this project:

```bash
rm -f "$HOME/.claude/beep_projects/$(pwd | tr '/' '_')"
```

Then tell the user: "Beep notifications are now **off** for this directory. Use `/noisily` to turn them back on."