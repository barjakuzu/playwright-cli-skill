# Playwright CLI Skill for Claude Code

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that supercharges browser automation with [Playwright CLI](https://github.com/microsoft/playwright-cli). Adds token-efficient workflows, structured QA processes, and ready-to-use recipes for common automation tasks.

## How This Relates to the Official Playwright CLI

This is **not** a fork or replacement of [microsoft/playwright-cli](https://github.com/microsoft/playwright-cli). The official CLI is the underlying binary — you still need it installed. This repo is a **Claude Code skill layer** on top of it.

Think of it like this:

```
┌─────────────────────────────────┐
│  This Skill (SKILL.md)          │  ← Tells Claude HOW to use the CLI efficiently
│  - Token rules                  │
│  - QA workflows                 │
│  - Recipes & patterns           │
├─────────────────────────────────┤
│  Playwright CLI (official)      │  ← The actual browser automation binary
│  github.com/microsoft/          │
│  playwright-cli                 │
├─────────────────────────────────┤
│  Playwright (engine)            │  ← Browser automation framework
└─────────────────────────────────┘
```

**Without this skill**, Claude Code can use Playwright CLI but wastes tokens on full DOM snapshots, reads unnecessary codegen output, and has no structured QA process.

**With this skill**, Claude follows token-efficient patterns, uses `eval` over `snapshot` for verification, suppresses codegen output, and follows a structured QA checklist.

## What This Adds

The official CLI gives Claude browser commands. This skill teaches Claude *when and how* to use them well.

| Feature | Official CLI alone | With this skill |
|---------|:-:|:-:|
| Token-efficient patterns | Claude figures it out | 7 enforced rules, codegen suppression |
| QA workflow | Ad-hoc | Structured inventory + checklists + signoff |
| Session management | Docs in README | Contextual recipes with named sessions |
| Storage/cookies | Raw API | Dedicated CLI commands documented |
| Request mocking | Raw API | Pattern library with conditional responses |
| Tracing & video | Available | Guided workflows with best practices |
| Test generation | Manual | Auto-generate from interactions |

### Token Savings

The skill enforces rules that cut token usage by ~30-40%:

- **Codegen suppression** — adds `{"codegen":"none"}` to skip JS code blocks from every command output
- **`eval` over `snapshot`** — verify with `eval "document.querySelector('.success')?.textContent"` instead of reading full DOM trees
- **Command chaining** — `fill e1 "text" && click e3` in one Bash call
- **Snapshot discipline** — only snapshot when you need element refs, never for verification

## Install

Copy the skill to your Claude Code skills directory:

```bash
# Clone
git clone https://github.com/anthropics/playwright-cli-skill.git

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
    ├── qa-workflow.md                # Structured QA process
    ├── test-generation.md            # Auto-generate Playwright tests
    ├── running-code.md               # Advanced: geolocation, permissions, waits, iframes
    ├── session-management.md         # Named sessions, concurrent automation
    ├── storage-state.md              # Cookies, localStorage, sessionStorage
    ├── tracing.md                    # Execution traces for debugging
    ├── video-recording.md            # Record sessions as WebM
    └── request-mocking.md            # Intercept, mock, block network requests
```

## Playwright CLI vs Chrome Extension — QA Benchmark

We ran both tools against the same intentionally buggy web page (21 planted bugs across 10 sections) to see which approach works better for AI-powered QA.

### Results

| Metric | Playwright CLI | Chrome Extension |
|--------|:-:|:-:|
| **True Positives** | **18 / 21** (86%) | 12 / 21 (57%) |
| **False Positives** | 9 | 9 |
| **Time** | **7m 32s** | 9m 13s |
| **Tokens** | **55K** | 75K |
| **Context Used** | **28%** | 38% |

### What Playwright CLI caught that Chrome missed

- Typo in success message ("Succesfully")
- Wizard step indicator desyncing on Back navigation
- Select All checkbox not updating with individual toggles
- Modal overlay click-to-close not working
- Missing `aria-live` on error messages
- Low contrast on status badges

### What Chrome Extension caught that Playwright missed

Nothing — Chrome found zero bugs that Playwright didn't also find.

### Why Playwright CLI Wins

1. **`eval` is surgical** — directly queries DOM properties, CSS values, aria attributes without loading full accessibility trees
2. **Viewport resizing is native** — `resize 375 812` tests responsive bugs reliably
3. **Lower token cost** — codegen suppression + eval-over-snapshot means more budget for actual testing
4. **Headless speed** — no UI rendering overhead, faster page interactions

### When Chrome Extension Is Better

| Scenario | Why |
|----------|-----|
| Testing pages behind SSO/OAuth | Extension uses your real browser session |
| Browser extension interactions | Only real Chrome can test extensions |
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
- New reference guides (e.g., PDF testing, multi-tab workflows)
- Token optimization techniques
- Additional QA benchmark results
- Bug fixes in skill documentation

## License

MIT
