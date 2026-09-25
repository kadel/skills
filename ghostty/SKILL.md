---
name: ghostty
description: Preview a Markdown file in a Ghostty split using AppleScript and glow. Use when the user asks to "render markdown in Ghostty", "preview markdown in a Ghostty split", or "show this file in a Ghostty pane".
metadata:
  version: "0.1.0"
---

## Purpose

Open a right split in Ghostty and preview a Markdown file with `glow`.

## Prerequisites

- **Ghostty 1.3.0 or later** must be running on macOS with AppleScript enabled
- **glow** CLI must be installed for markdown rendering (`brew install glow`)

## Commands

### render-markdown-split

Open a right split in the current Ghostty window and render a markdown file using `glow`.

#### Arguments

- `file` (required): Path to the markdown file to render

#### Workflow

1. Resolve the markdown file path to an absolute path
2. Verify the file exists and has a `.md` extension
3. Pass the absolute path as an argument to AppleScript. AppleScript shell-quotes it before sending the command to Ghostty:

```bash
ABSOLUTE_FILE_PATH='/absolute/path/to/file.md'
osascript - "$ABSOLUTE_FILE_PATH" <<'APPLESCRIPT'
on run argv
    set filePath to item 1 of argv
    set commandText to "glow -p " & (quoted form of filePath)

    tell application "Ghostty"
        set currentTerm to focused terminal of selected tab of front window
        set previewTerm to split currentTerm direction right
        delay 0.5
        input text commandText to previewTerm
        send key "enter" to previewTerm
    end tell
end run
APPLESCRIPT
```

#### Important Notes

- The `delay 0.5` is necessary to allow the new split to initialize before sending input
- Always use absolute paths for the file to avoid working directory issues in the new split
- Pass the path as an `osascript` argument; do not interpolate it into shell or AppleScript source
- The `-p` flag enables pager mode in glow for scrollable output
- If `glow` is not installed, inform the user and suggest `brew install glow`

#### Error Handling

- If the file does not exist, report the error to the user — do not run the AppleScript
- If `osascript` returns an error about `focused terminal`, Ghostty may not be the active application — advise the user to focus Ghostty first
