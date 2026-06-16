---
name: sxt
description: "Use when working alongside a user in their browser: reading pages they have open, extracting page content, inspecting DOM state, performing lightweight page interaction, or managing tabs and windows. Not intended for large browser automation workflows."
---

# sxt — Browser Companion

macOS only. Requires scriptable browser (default: Orion; env `BROWSER_APP` or `-b` flag to override with safari).
Enable "Allow JavaScript from Apple Events" in browser's Develop menu.


## Commands

```bash
# Orient
sxt list -o title,url           # what's open (clean)
sxt list                        # full: window, front, tab, current, title, url

# Read
sxt read                        # current tab → markdown
sxt read -w <id> -t <n>        # specific tab; IDs/indices from sxt list

# Tabs
sxt open <url>                  # new tab
sxt nav <url>                   # navigate current tab
sxt nav <url> -w <id> -t <n>   # navigate specific tab
sxt close                       # close current tab
sxt close -w <id> -t <n>

# JS
sxt js "<code>"                 # run in current tab, return result
sxt js "<code>" -w <id> -t <n>
```

## JS — skip read when you need targeted data

`sxt` intentionally exposes a small command surface. Many page-specific operations should be performed through `js` rather than requiring dedicated commands.

```bash
# Page identity
sxt js "document.title"
sxt js "window.location.href"
sxt js "document.querySelector('meta[name=description]').content"

# Raw HTML (when you need DOM not markdown)
sxt js "document.documentElement.outerHTML"

# Targeted text (cheaper than full read)
sxt js "document.querySelector('article').innerText"
sxt js "document.querySelector('h1').innerText"
sxt js "document.querySelector('#price').innerText"

# Structured extraction
sxt js "JSON.stringify(Array.from(document.querySelectorAll('a')).map(a=>({text:a.innerText.trim(),href:a.href})))"
sxt js "JSON.stringify(Array.from(document.querySelectorAll('table tr')).map(r=>Array.from(r.cells).map(c=>c.innerText.trim())))"
```

Objects need `JSON.stringify()` or you get `[object Object]`.

Prefer concise JavaScript snippets when the task is specific to a page's DOM.

## Gotchas

- **Suspended tab:** WebKit freezes background tabs; `read`/`js` fail. `sxt nav <url>` to reload in front window.
- **JS not enabled:** enable "Allow JavaScript from Apple Events" in Develop menu.
