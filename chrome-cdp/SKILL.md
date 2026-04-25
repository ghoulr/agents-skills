---
name: chrome-cdp
description: Interact with local Chrome browser session (only on explicit user approval after being asked to inspect, debug, or interact with a page open in Chrome)
---

# Chrome CDP

Lightweight Chrome DevTools Protocol CLI. Connects directly via WebSocket — no Puppeteer, works with 100+ tabs, instant connection.

## Prerequisites

- Chrome (or Chromium, Brave, Edge, Vivaldi) with remote debugging enabled: open `chrome://inspect/#remote-debugging` and toggle the switch
- Node.js 22+ (uses built-in WebSocket)
- If your browser's `DevToolsActivePort` is in a non-standard location, set `CDP_PORT_FILE` to its full path

## Commands

All commands use `scripts/cdp.mjs`. The `<target>` is a **unique** targetId prefix from `list`; copy the full prefix shown in the `list` output (for example `6BE827FA`). The CLI rejects ambiguous prefixes.

All page-scoped commands also accept `--frame <iframeTargetId>` to route the command into a cross-origin iframe (see **Cross-origin iframe support** below).

### List open pages

```bash
scripts/cdp.mjs list
```

### List iframe targets in a page

```bash
scripts/cdp.mjs iframes <target>
```

Discovers all iframes (including cross-origin) belonging to the page. Shows the iframe target IDs needed for `--frame`. Nested iframes are indented under their parent.

### Take a screenshot

```bash
scripts/cdp.mjs shot <target> [file]    # default: screenshot-<target>.png in runtime dir
```

Captures the **viewport only**. Scroll first with `eval` if you need content below the fold. Output includes the page's DPR and coordinate conversion hint (see **Coordinates** below).

### Accessibility tree snapshot

```bash
scripts/cdp.mjs snap <target>
```

### Evaluate JavaScript

```bash
scripts/cdp.mjs eval <target> <expr>
```

> **Watch out:** avoid index-based selection (`querySelectorAll(...)[i]`) across multiple `eval` calls when the DOM can change between them (e.g. after clicking Ignore, card indices shift). Collect all data in one `eval` or use stable selectors.

### Other commands

```bash
scripts/cdp.mjs html    <target> [selector]   # full page or element HTML
scripts/cdp.mjs nav     <target> <url>         # navigate and wait for load
scripts/cdp.mjs net     <target>               # resource timing entries
scripts/cdp.mjs click   <target> <selector>    # click element by CSS selector
scripts/cdp.mjs clickxy <target> <x> <y>       # click at CSS pixel coords
scripts/cdp.mjs type    <target> <text>         # Input.insertText at current focus; works in cross-origin iframes unlike eval
scripts/cdp.mjs loadall <target> <selector> [ms]  # click "load more" until gone (default 1500ms between clicks)
scripts/cdp.mjs evalraw <target> <method> [json]  # raw CDP command passthrough
scripts/cdp.mjs open    [url]                  # open new tab (each triggers Allow prompt)
scripts/cdp.mjs stop    [target]               # stop daemon(s)
```

## Coordinates

`shot` saves an image at native resolution: image pixels = CSS pixels × DPR. CDP Input events (`clickxy` etc.) take **CSS pixels**.

```
CSS px = screenshot image px / DPR
```

`shot` prints the DPR for the current page. Typical Retina (DPR=2): divide screenshot coords by 2.

## Cross-origin iframe support

When a page contains cross-origin iframes, commands like `eval`/`snap`/`html`/`click` only operate on the main page context by default. To interact with an iframe:

1. Run `scripts/cdp.mjs iframes <target>` to discover iframe target IDs.
2. Append `--frame <iframeTargetId>` to any page-scoped command.

```bash
# List iframes (shows target IDs and nested structure)
scripts/cdp.mjs iframes <target>

# Evaluate JS inside a cross-origin iframe
scripts/cdp.mjs eval <target> --frame <iframeTargetId> "document.title"

# Accessibility snapshot of an iframe
scripts/cdp.mjs snap <target> --frame <iframeTargetId>

# Get HTML from an iframe
scripts/cdp.mjs html <target> --frame <iframeTargetId> "body"

# Click an element inside an iframe
scripts/cdp.mjs click <target> --frame <iframeTargetId> "button.submit"
```

The daemon auto-attaches to the iframe target on first `--frame` use and caches the session. Subsequent commands to the same iframe are instant.

For **same-origin** iframes, you can access content directly without `--frame`:
```bash
scripts/cdp.mjs eval <target> "document.querySelector('iframe').contentDocument.title"
```

## Tips

- Prefer `snap` over `html` for page structure — it's more concise.
- Use `type` (not eval) to enter text in cross-origin iframes — `click`/`clickxy` to focus first, then `type`.
- Chrome shows an "Allow debugging" modal once per tab on first access. A background daemon keeps the session alive so subsequent commands need no further approval. Daemons auto-exit after 20 minutes of inactivity.
- Use `iframes` to discover cross-origin iframes that don't appear in the main page's accessibility tree.
