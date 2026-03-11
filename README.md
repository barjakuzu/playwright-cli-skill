# Playwright CLI Skill for Claude Code

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that adds token-efficient workflows on top of [Playwright CLI](https://github.com/microsoft/playwright-cli) for browser automation.

## What's Different from the Official Skill

The official `playwright-cli install --skills` gives Claude a SKILL.md + 7 reference files. This repo uses the **same reference files** but replaces the SKILL.md with one that adds:

1. **Codegen suppression** — auto-creates `{"codegen":"none"}` config to skip JS code blocks from every command output
2. **Eval-over-snapshot rules** — enforces `eval` for verification instead of expensive DOM snapshots
3. **Command chaining** — instructs Claude to use `&&` to reduce tool call overhead
4. **Snapshot discipline** — only snapshot when element refs are needed, never for verification
5. **Arrow function ban in eval** — prevents parse errors from `=>` and `...` syntax
6. **Drag-and-drop workaround** — uses JS simulation via `eval` since built-in `drag` is buggy
7. **Named session guidance** — isolate projects with `PLAYWRIGHT_CLI_SESSION` or `-s=name`

The reference files (`references/*.md`) are identical to what ships with the official install.

## Benchmark

We tested custom skill, official skill, and Chrome extension against a page with 21 intentional bugs across 10 sections. All runs used **sandbox mode** — no memory, no prior context, clean temp directory, clean HTML with no code comments hinting at bugs.

| Metric | Custom Skill | Official Skill | Chrome Extension |
|--------|:-:|:-:|:-:|
| **True Positives** | **19 / 21 (90%)** | **19 / 21 (90%)** | 13 / 21 (62%) |
| **False Positives** | **3** | 9 | 12 |
| **Precision** | **86%** | 68% | 52% |
| **Time** | 3m 29s | 3m 30s | 6m 29s |
| **Tokens** | **43K** | 52K | 53K |

### Custom vs Official Skill

Both found the same 19 bugs and missed the same 2. The difference is noise: the official skill reported 9 false positives (missing validation, misleading cursor styles, aria-label suggestions, etc.). The custom skill's token rules kept it focused — only 3 false positives, 17% fewer tokens.

**The custom skill doesn't find more bugs. It finds them with less noise and fewer tokens.**

### Playwright CLI vs Chrome Extension

Chrome extension missed 8 bugs that Playwright CLI caught:

| Missed Bug | Why |
|------------|-----|
| Typo "Succesfully" in success message | Not detected |
| Wizard step indicator desyncs on Back | Reported wrong issue |
| Modal overlay click-to-close broken | Not detected |
| Modal focus trap missing | Reported different bug |
| Badge low contrast (#bbf7d0 on #dcfce7) | Not detected |
| Error div missing aria-live | Reported wrong element |
| Counter desync on double-load | Reported different bug |
| Step 2 data loss on Back | Not detected |

Chrome also reported 12 false positives including factually incorrect claims like "no drag event handlers attached."

**Why Playwright CLI wins:** `eval` gives surgical DOM access (properties, computed styles, aria attributes). Viewport resizing is native. Source code analysis catches logic bugs. Lower token cost leaves more budget for actual testing.

**When Chrome Extension is better:** pages behind SSO/OAuth (uses your real session), testing browser extensions, visual verification by a human, quick one-off checks with no setup.

## Install

```bash
git clone https://github.com/barjakuzu/playwright-cli-skill.git
cp -r playwright-cli-skill ~/.claude/skills/playwright-cli
```

Requires [Playwright CLI](https://github.com/microsoft/playwright-cli):

```bash
npm install -g @anthropic-ai/playwright-cli
```

## Usage

```
Use /playwright-cli to test my web app at http://localhost:3000
```

The skill activates automatically when you ask Claude to navigate websites, fill forms, take screenshots, or test web apps.

## Quick Reference

```bash
# Core workflow
playwright-cli open https://example.com     # Open browser
playwright-cli snapshot                      # Get element refs
playwright-cli fill e1 "text" && click e3    # Interact
playwright-cli eval "document.title"         # Verify (cheap)
playwright-cli close                         # Done

# QA testing
playwright-cli resize 375 812               # Test mobile
playwright-cli screenshot --filename=m.png   # Capture evidence
playwright-cli eval "document.querySelector('.badge')?.textContent"

# Sessions
playwright-cli -s=auth open https://app.com  # Named session
playwright-cli close-all                      # Cleanup

# Request mocking
playwright-cli route "**/api/users" --body='[{"id":1}]' --content-type=application/json
```

## Contributing

PRs welcome for token optimization techniques, benchmark results, and new reference guides.

## License

MIT
