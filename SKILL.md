---
name: playwright-cli
description: Automates browser interactions for web testing, form filling, screenshots, and data extraction. Use when the user needs to navigate websites, interact with web pages, fill forms, take screenshots, test web applications, or extract information from web pages.
allowed-tools: Bash(playwright-cli:*)
---

# Browser Automation with playwright-cli

## Setup: disable codegen for token savings

Create `~/.playwright/cli.config.json` if it doesn't exist:
```json
{ "codegen": "none" }
```
This suppresses the `### Ran Playwright code` JS blocks from every command output, saving significant tokens.

## Token-efficiency rules (CRITICAL)

1. **Do NOT read snapshot YAML files** unless you need element refs for the next interaction. The snapshot output from each command already shows the page URL and title inline.
2. **Use `eval` for verification**, not snapshots. Example: `playwright-cli eval "document.querySelector('.flash')?.textContent"` is far cheaper than snapshot + Read.
3. **Chain commands with `&&`** to reduce tool call overhead.
4. **Only snapshot when you need element refs** (e.g., before clicking, filling, or selecting elements by ref ID).
5. **Use `eval` for data extraction** instead of reading a full snapshot YAML.
6. **Never use arrow functions (`=>`) or spread (`...`) in `eval`** — they cause parse errors. Use `function(){}` and `Array.from()` instead.
7. **The `drag` command has a bug** — use `eval` with JS drag simulation for drag-and-drop tasks.
8. **Use named sessions** with `PLAYWRIGHT_CLI_SESSION` env var or `-s=name` to isolate projects.
9. **Use `playwright-cli show`** to open a visual dashboard for monitoring background automation.

## Core workflow

```bash
# Open and navigate
playwright-cli open https://example.com

# Snapshot ONLY to discover element refs (e1, e2, etc.)
playwright-cli snapshot
# Read the snapshot file ONLY when you need refs

# Interact using refs
playwright-cli fill e5 "text" && playwright-cli click e3

# Verify with eval (NOT snapshot)
playwright-cli eval "document.title"
playwright-cli eval "window.location.href"
playwright-cli eval "document.querySelector('h1')?.textContent"

# Close when done
playwright-cli close
```

## When to snapshot vs eval

| Need | Use |
|------|-----|
| Find element refs to click/fill/select | `snapshot` → Read the YAML |
| Verify text appeared | `eval "document.querySelector('.selector')?.textContent"` |
| Check URL | `eval "window.location.href"` |
| Check page title | `eval "document.title"` |
| Extract table data | `eval "JSON.stringify(...)"` |
| Check element state | `eval "document.querySelector('#cb').checked"` |
| Check selected option | `eval "document.querySelector('select').value"` |

## Commands

### Interaction
```bash
playwright-cli open [url]          # open browser, optionally navigate
playwright-cli goto <url>          # navigate to URL
playwright-cli click <ref>         # click element
playwright-cli dblclick <ref>      # double-click
playwright-cli fill <ref> <text>   # fill input field
playwright-cli type <text>         # type into focused element
playwright-cli press <key>         # press key (Enter, ArrowDown, etc.)
playwright-cli select <ref> <val>  # select dropdown option
playwright-cli check <ref>         # check checkbox/radio
playwright-cli uncheck <ref>       # uncheck checkbox
playwright-cli hover <ref>         # hover over element
playwright-cli drag <start> <end>  # drag and drop
playwright-cli upload <file>       # upload file
```

### Navigation
```bash
playwright-cli go-back / go-forward / reload
```

### Inspection
```bash
playwright-cli snapshot                    # get element refs (save to YAML)
playwright-cli eval "<js expression>"      # run JS, get result (PREFERRED for verification)
playwright-cli eval "<fn>" <ref>           # run JS on specific element
playwright-cli screenshot [--filename=f]   # save screenshot
playwright-cli console                     # read console messages
playwright-cli network                     # list network requests
```

### Tabs
```bash
playwright-cli tab-list / tab-new [url] / tab-close [idx] / tab-select <idx>
```

### Browser options
```bash
playwright-cli open --browser=chrome|firefox|webkit|msedge
playwright-cli open --headed          # visible browser window
playwright-cli open --persistent      # persistent profile
playwright-cli resize <w> <h>        # resize viewport
playwright-cli close                  # close browser
playwright-cli close-all             # close all sessions
```

## Example: Form submission (token-efficient)

```bash
# Open and get refs
playwright-cli open https://example.com/form
playwright-cli snapshot
# Read snapshot YAML to find refs, then:
playwright-cli fill e1 "user@example.com" && playwright-cli fill e2 "password" && playwright-cli click e3
# Verify with eval, NOT snapshot
playwright-cli eval "document.querySelector('.success')?.textContent"
playwright-cli close
```

## Example: Data extraction

```bash
playwright-cli open https://example.com/table
# Extract data directly with eval — no snapshot needed
# NOTE: no arrow functions or spread syntax in eval!
playwright-cli eval "JSON.stringify(Array.from(document.querySelectorAll('#table1 tbody tr')).map(function(r){return Array.from(r.cells).map(function(c){return c.textContent})}))"
playwright-cli close
```

## Example: Verify without snapshot

```bash
# After clicking a login button:
playwright-cli eval "window.location.href"                           # check redirect
playwright-cli eval "document.querySelector('.flash')?.textContent"  # check flash message
playwright-cli eval "document.querySelector('#finish h4')?.textContent"  # check loaded text
```

## Specific tasks

* **Request mocking** [references/request-mocking.md](references/request-mocking.md)
* **Running Playwright code** [references/running-code.md](references/running-code.md)
* **Browser session management** [references/session-management.md](references/session-management.md)
* **Storage state** [references/storage-state.md](references/storage-state.md)
* **Test generation** [references/test-generation.md](references/test-generation.md)
* **Tracing** [references/tracing.md](references/tracing.md)
* **Video recording** [references/video-recording.md](references/video-recording.md)

