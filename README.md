# orioncli

A minimal CLI for controlling [Orion browser](https://browser.kagi.com) from the shell or an agent.

Orion exposes AppleScript and `do JavaScript` for browser automation. This wraps those into five orthogonal commands with consistent flags and tab-separated output — so the interesting workflows stay in the shell, not the tool.

---

## Prerequisites

In Orion: **Develop → Allow JavaScript from Apple Events**

Without this, `read` and `js` will error with instructions.

---

## Example

```sh
# What's open?
orion list

# Read the current tab as markdown
orion read

# Read a specific tab
orion list                          # find window ID and tab index
orion read -w 29178 -t 3

# Run JS in a tab
orion js "document.title"
orion js "JSON.stringify([...document.querySelectorAll('h2')].map(h=>h.innerText))"

# Navigate and read
orion nav https://example.com
orion read

# Open a new tab
orion open https://example.com
```

---

## Commands

```
orion list              list all windows and tabs (tab-separated)
orion read              get page content as markdown
orion js <code>         execute JavaScript, return result
orion nav <url>         navigate a tab to URL
orion open <url>        open URL in a new tab
```

**Flags (read, js, nav):**

```
-w <id>    window ID from list (default: front window)
-t <n>     tab index, 1-based (default: current tab)
```

**Flags (list):**

```
-o <fields>   comma-separated output fields, no header (pipeline mode)
              fields: window, front, tab, current, title, url
```

URL for `nav` and `open` may be piped via stdin.

---

## list output

```
window  front  tab  current  title                   url
29178   true   1    false    Example Domain           https://example.com/
29178   true   2    true     GitHub                   https://github.com/
29224   false  1    false    Hacker News              https://news.ycombinator.com/
```

`window` is a stable ID — it doesn't change as windows move between front and back. Use it with `-w` for reliable targeting across commands.

`front` and `current` indicate the active window and active tab within each window.

---

## Composability

```sh
# All URLs open right now
orion list -o url

# Titles and URLs for one window
orion list -o tab,title,url | grep "^29178"

# Open every GitHub URL in a new tab
orion list -o url | grep github.com | while read -r url; do orion open "$url"; done

# Pick a tab to read with fzf
orion list -o window,tab,title \
  | fzf \
  | awk '{print "-w", $1, "-t", $2}' \
  | xargs orion read

# Send current page to an API
orion read | curl -s -X POST https://api.example.com/summarize -d @-

# Extract all links from a page
orion js "JSON.stringify([...document.querySelectorAll('a')].map(a=>a.href))"

# Navigate from stdin
echo "https://example.com" | orion nav
```

---

## Notes

**`read` and `js` require a visible tab.** WebKit suspends JavaScript in background windows to save memory. If you see:

```
error: tab is suspended (WebKit freezes background tabs); use 'nav <url>' to load it in the front window first
```

Load the page in the front window first:

```sh
orion nav https://example.com      # loads in front window, current tab
orion read                         # reads it
```

**`read` returns rendered content.** It gets the post-JavaScript DOM, not raw source — so SPAs and dynamically loaded content are included. Use `orion js "document.documentElement.outerHTML"` if you need the raw HTML.

---

## Install

```sh
go install github.com/PhilShaughnes/orioncli@latest
```

Or build from source:

```sh
git clone https://github.com/PhilShaughnes/orioncli
cd orioncli
go build -o orion .
```
