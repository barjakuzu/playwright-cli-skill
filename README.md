# Playwright CLI Skill for Claude Code

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that supercharges browser automation with [Playwright CLI](https://github.com/microsoft/playwright-cli). Adds token-efficient workflows and ready-to-use recipes for common automation tasks.

## How This Relates to the Official Playwright CLI

This is **not** a fork or replacement of [microsoft/playwright-cli](https://github.com/microsoft/playwright-cli). The official CLI is the underlying binary — you still need it installed. This repo is a **Claude Code skill layer** on top of it.

```
┌─────────────────────────────────┐
│  This Skill (SKILL.md)          │  ← Tells Claude HOW to use the CLI efficiently
│  - Token-efficiency rules       │
│  - Eval-over-snapshot patterns  │
│  - Recipes & references         │
├─────────────────────────────────┤
│  Playwright CLI (official)      │  ← The actual browser automation binary
│  github.com/microsoft/          │
│  playwright-cli                 │
├─────────────────────────────────┤
│  Playwright (engine)            │  ← Browser automation framework
└─────────────────────────────────┘
```

**Without this skill**, Claude Code can use Playwright CLI but wastes tokens on full DOM snapshots, reads unnecessary codegen output, and takes an unstructured approach to testing.

**With this skill**, Claude follows token-efficient patterns, uses `eval` over `snapshot` for verification, and suppresses codegen output.

## What This Adds

The official `playwright-cli install --skills` gives Claude browser commands and reference docs. This skill adds **7 token-efficiency rules** in the main SKILL.md that change how Claude uses those commands:

| What | Official skill alone | With this skill |
|------|:-:|:-:|
| Codegen output | Included (extra tokens) | Suppressed via config |
| Verification | Snapshots (expensive) | `eval` expressions (cheap) |
| Command calls | One per tool call | Chained with `&&` |
| Arrow functions in eval | Used (causes parse errors) | Explicitly banned |
| Drag-and-drop | Built-in `drag` (buggy) | JS simulation via `eval` |
| Snapshot reads | Frequent | Only when refs needed |

### Token Savings

The rules cut token usage by ~17-20%:

- **Codegen suppression** — `{"codegen":"none"}` skips JS code blocks from every command output
- **`eval` over `snapshot`** — `eval "document.querySelector('.success')?.textContent"` instead of reading full DOM trees
- **Command chaining** — `fill e1 "text" && click e3` in one Bash call
- **Snapshot discipline** — only snapshot when you need element refs, never for verification

## Install

```bash
# Clone
git clone https://github.com/barjakuzu/playwright-cli-skill.git

# Copy to Claude Code skills
cp -r playwright-cli-skill ~/.claude/skills/playwright-cli
```

### Prerequisites

Install Playwright CLI ([official repo](https://github.com/microsoft/playwright-cli)):

```bash
npm install -g @anthropic-ai/playwright-cli
```

## Usage

In Claude Code, just say:

```
Use /playwright-cli to test my web app at http://localhost:3000
```

Or invoke it directly:

```
/playwright-cli
```

The skill activates automatically when you ask Claude to navigate websites, fill forms, take screenshots, or test web apps.

## Skill Structure

```
~/.claude/skills/playwright-cli/
├── SKILL.md                          # Main skill definition + token rules
└── references/
    ├── test-generation.md            # Auto-generate Playwright tests
    ├── running-code.md               # Advanced: geolocation, permissions, waits, iframes
    ├── session-management.md         # Named sessions, concurrent automation
    ├── storage-state.md              # Cookies, localStorage, sessionStorage
    ├── tracing.md                    # Execution traces for debugging
    ├── video-recording.md            # Record sessions as WebM
    └── request-mocking.md            # Intercept, mock, block network requests
```

> **Note:** The 7 reference files ship with the official `playwright-cli install --skills`. The custom value is in the SKILL.md rules, not the references.

## Custom Skill vs Official Skill — QA Benchmark

We tested both skill versions against a page with 21 intentional bugs (10 sections, clean HTML with no code comments hinting at bugs). Both ran in **sandbox mode** — no memory, no prior context, clean temp directory.

### Results

| Metric | Custom Skill | Official Skill |
|--------|:-:|:-:|
| **True Positives** | **19 / 21 (90%)** | **19 / 21 (90%)** |
| **False Positives** | **3** | 9 |
| **Precision** | **86%** | 68% |
| **Time** | 3m 29s | 3m 30s |
| **Tokens** | **43K** | 52K |

Both found the same 19 real bugs and missed the same 2 (step data loss on back navigation, counter desync on rapid loading).

### Where custom wins: precision

The official skill reported 9 false positives (missing validation, misleading cursor styles, aria-label on close buttons, etc.) — valid observations but not actual bugs. The custom skill's token-efficiency rules kept it focused, producing only 3 false positives.

### Where they're equal: recall

Same 90% recall. The token rules don't help or hurt bug detection — they just reduce noise and cost.

### Honest takeaway

The custom skill doesn't find more bugs. It finds them **with less noise and fewer tokens**. If you care about precision and token cost, use this skill. If you just want bug coverage, the official skill works fine.

## Playwright CLI vs Chrome Extension

We also compared Playwright CLI (with custom skill) against the [Claude-in-Chrome](https://chromewebstore.google.com/detail/claude-in-chrome/flkfmibmamgolkkoeapdmcnikmhpmjgd) browser extension for QA, using the same test page and prompt. All runs used sandbox mode (no memory, clean session).

### Results

| Metric | Playwright CLI (custom) | Playwright CLI (official) | Chrome Extension |
|--------|:-:|:-:|:-:|
| **True Positives** | **19 / 21 (90%)** | **19 / 21 (90%)** | 13 / 21 (62%) |
| **False Positives** | **3** | 9 | 12 |
| **Precision** | **86%** | 68% | 52% |
| **Recall** | **90%** | **90%** | 62% |
| **Time** | 3m 29s | 3m 30s | 6m 29s |
| **Tokens** | **43K** | 52K | 53K |

### What Chrome Extension missed (8 bugs)

| Bug | Why missed |
|-----|-----------|
| Typo "Succesfully" in success message | Not detected at all |
| Wizard step indicator desyncs on Back | Reported different issue ("all steps visible") |
| Modal overlay click-to-close broken | Not detected |
| Modal focus trap missing | Reported "focus goes to body on close" (different bug) |
| Badge low contrast (#bbf7d0 on #dcfce7) | Not detected |
| Error div missing aria-live | Reported wrong element ("status badge") |
| Counter desync on double-load | Reported "counter not accessible" (different bug) |
| Step 2 data loss on Back | Not detected |

### Chrome Extension false positives (12)

Reported issues not in the bug manifest: form uses GET method, status header missing sort indicator, "no drag event handlers" (factually wrong), all wizard steps visible, no field validation, missing role=dialog, close button no aria-label, focus goes to body on close, counter not programmatically accessible, grid uses fixed columns, phone input not type=tel, status badge missing aria-live.

### Why Playwright CLI wins for QA

1. **`eval` is surgical** — directly queries DOM properties, CSS values, computed styles, aria attributes
2. **Viewport resizing is native** — `resize 375 812` tests responsive bugs reliably
3. **Source analysis** — can read and analyze full page source to find logic bugs in JavaScript
4. **Lower token cost** — codegen suppression + eval-over-snapshot means more budget for actual testing

### When Chrome Extension is better

| Scenario | Why |
|----------|-----|
| Pages behind SSO/OAuth | Uses your real browser session |
| Browser extension testing | Only real Chrome can test extensions |
| Visual verification by human | You see what Claude sees in real-time |
| Quick one-off checks | No setup, just point at the tab |

## Quick Reference

### Core Workflow

```bash
playwright-cli open https://example.com     # Open browser
playwright-cli snapshot                      # Get element refs
playwright-cli fill e1 "text" && click e3    # Interact
playwright-cli eval "document.title"         # Verify (cheap)
playwright-cli close                         # Done
```

### QA Testing

```bash
playwright-cli open https://myapp.com
playwright-cli snapshot

# Test form
playwright-cli fill e1 "test@email.com" && playwright-cli click e5
playwright-cli eval "document.querySelector('.success')?.textContent"

# Test responsive
playwright-cli resize 375 812
playwright-cli screenshot --filename=mobile.png
playwright-cli resize 1920 1080

# Check accessibility
playwright-cli eval "document.querySelector('label[for]')?.htmlFor === document.querySelector('input')?.id"
```

### Session Management

```bash
playwright-cli -s=auth open https://app.com/login    # Named session
playwright-cli -s=public open https://app.com         # Isolated session
playwright-cli close-all                               # Cleanup
```

### Request Mocking

```bash
playwright-cli route "**/api/users" --body='[{"id":1}]' --content-type=application/json
playwright-cli route "**/*.jpg" --status=404
playwright-cli unroute
```

## Contributing

PRs welcome for:
- Token optimization techniques
- Additional QA benchmark results
- New reference guides
- Bug fixes

## License

MIT
