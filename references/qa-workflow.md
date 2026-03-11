# QA Workflow with playwright-cli

Structured QA process for verifying web applications using playwright-cli. Adapted for Claude Code's Bash + Read tool model.

## QA Inventory (Build Before Testing)

Before testing, build a coverage list from three sources:

1. **User requirements** — what was requested
2. **Implemented features** — what you actually built or see on the page
3. **Claims you'll make** — what you'll tell the user works

Every item in any of those three must map to at least one QA check.

### Inventory template

```
Controls: [list every interactive element]
States: [list state changes each control causes]
Functional checks: [what to verify for each control-state pair]
Visual checks: [what to inspect at each state]
Exploratory: [2+ off-happy-path scenarios]
```

## Functional QA

### Checklist

1. **Use real interactions** — `click`, `fill`, `type`, `press`, `select` via refs. Not `eval` or `run-code`.
2. **Verify at least one end-to-end flow** — e.g., open page, fill form, submit, check result.
3. **Confirm visible results** — use `snapshot` to check text/state, not just internal values.
4. **Test full cycles for stateful controls** — initial state -> changed -> back to initial.
5. **Cover every visible control** at least once before signoff.
6. **Short exploratory pass** — 2+ scenarios beyond the happy path (wrong input, double-click, back button, etc.).

### Pattern: Form testing

```bash
# Open and inspect
playwright-cli open https://example.com/form
playwright-cli snapshot

# Fill fields using refs from snapshot
playwright-cli fill e1 "valid@email.com"
playwright-cli fill e2 "ValidPassword123"
playwright-cli click e3  # submit
playwright-cli snapshot  # verify success

# Test invalid input
playwright-cli goto https://example.com/form
playwright-cli snapshot
playwright-cli fill e1 "invalid-email"
playwright-cli click e3  # submit
playwright-cli snapshot  # verify error message

# Test empty submission
playwright-cli goto https://example.com/form
playwright-cli snapshot
playwright-cli click e3  # submit without filling
playwright-cli snapshot  # verify validation
```

## Visual QA

### Checklist

1. **Screenshot at key states** — before interaction, after interaction, error states.
2. **Check viewport fit** — no unintended clipping, overflow, or scrollbars.
3. **Inspect initial view first** — what's visible without scrolling.
4. **Test responsive** if applicable — use `resize` to check breakpoints.
5. **Look for:** clipping, overflow, distortion, alignment issues, illegible text, weak contrast, broken layering.
6. **Judge aesthetics** — UI should feel intentional and coherent.

### Pattern: Visual verification

```bash
# Capture initial state
playwright-cli screenshot --filename=initial.png
# Read the screenshot to inspect visually
# (Use Claude Code's Read tool on the screenshot file)

# Interact and capture result
playwright-cli click e5
playwright-cli screenshot --filename=after-click.png

# Check responsive layout
playwright-cli resize 375 812  # iPhone viewport
playwright-cli screenshot --filename=mobile.png
playwright-cli resize 1920 1080  # back to desktop
```

### Viewport fit check

```bash
playwright-cli eval "JSON.stringify({innerWidth: window.innerWidth, innerHeight: window.innerHeight, scrollWidth: document.documentElement.scrollWidth, scrollHeight: document.documentElement.scrollHeight, canScrollX: document.documentElement.scrollWidth > document.documentElement.clientWidth, canScrollY: document.documentElement.scrollHeight > document.documentElement.clientHeight})"
```

**Signoff fails if:**
- Any required visible region is clipped or pushed outside the viewport
- Fixed-shell interfaces require scrolling to reach primary controls
- Screenshot shows visible defects that numeric checks miss

## Signoff Criteria

Before declaring QA complete:

- [ ] Functional path passed with real user interactions
- [ ] Coverage matches the QA inventory — note any intentional exclusions
- [ ] Visual QA covered the whole relevant interface
- [ ] Each user-visible claim has a matching screenshot
- [ ] Viewport-fit checks passed for intended initial view
- [ ] UI is visually coherent (not just functional)
- [ ] Exploratory pass completed — note what it covered
- [ ] Brief negative confirmation: list defect classes checked for and not found

## Common Failure Modes

| Symptom | Fix |
|---------|-----|
| `ref not found` after interaction | Re-snapshot: `playwright-cli snapshot` |
| Page didn't navigate after click | Check if click opened new tab: `playwright-cli tab-list` |
| Form submit has no visible effect | Check console: `playwright-cli console` |
| Screenshot is blank/empty | Page may still be loading — add `playwright-cli eval "document.readyState"` check |
| Element not interactable | Try `playwright-cli hover eX` first, or check if element is behind an overlay |

## Session Management During QA

- Keep the browser alive across all QA steps — don't close between checks
- Re-snapshot after every navigation or significant DOM change
- Use `playwright-cli console` and `playwright-cli network` for debugging failures
- Only `playwright-cli close` when QA is truly complete
