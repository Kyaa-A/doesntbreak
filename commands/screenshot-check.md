---
description: Render a page in a real headless browser at four widths and prove whether it horizontally overflows. The measured counterpart to /doesntbreak:audit. Saves before/after screenshots.
argument-hint: [local .html file or http(s) URL to check, defaults to the project's examples]
---

You are running a **measured** responsive check. Unlike `/doesntbreak:audit`, which
reasons from source, this command loads the target in a real headless browser at
several widths and reports whether the page actually overflows horizontally. A
page-level horizontal scrollbar is the bug this hunts for.

## Target

Target: `$ARGUMENTS`

Resolve it as follows:

- Starts with `http://` or `https://`: load the URL directly (the user's dev server).
- Otherwise: treat it as a local file path. The Playwright MCP blocks the `file://`
  protocol, so serve the file's directory over HTTP and load it from localhost.
  Start a static server in that directory (for example
  `python3 -m http.server 8765 --bind 127.0.0.1`) in the background, navigate to
  `http://127.0.0.1:8765/<filename>`, and stop the server when the run finishes. The
  npx fallback in Path B opens `file://` directly and does not need a server.
- If no argument is given: check `examples/*.html` if they exist; otherwise ask the
  user for a target and stop.
- If the path does not resolve to a real file and is not a URL, say so plainly and
  stop. Do not invent results.

## Widths

Check these four, in order: **320, 375, 768, 1440** (px). They are the smallest
phones, common phones, the awkward tablet middle, and a desktop control width.

## Browser tooling

Prefer a connected Playwright MCP browser. If none is available, fall back to a
throwaway `npx playwright` script. If neither works, report that a browser tool is
required (Playwright MCP, or Node with `npx playwright`) and stop.

### Path A: Playwright MCP (preferred)

For each width: resize the viewport to that width (height 800 is fine), navigate to
the target, wait for load, then evaluate this in the page and record the result:

```js
() => {
  const docW = document.documentElement.scrollWidth;
  const viewW = window.innerWidth;
  const overflow = docW - viewW;
  let offender = null;
  if (overflow > 1) {
    let worst = null, worstRight = viewW;
    for (const el of document.querySelectorAll('*')) {
      const r = el.getBoundingClientRect();
      if (r.width > 0 && r.right > worstRight + 1) { worst = el; worstRight = r.right; }
    }
    if (worst) offender = {
      tag: worst.tagName.toLowerCase(),
      id: worst.id || null,
      cls: (typeof worst.className === 'string' && worst.className) || null,
      right: Math.round(worstRight),
    };
  }
  return { docW, viewW, overflow, offender };
}
```

Then take a full-page screenshot for that width. `overflow > 1` (a one-pixel
tolerance for sub-pixel rounding) means the page scrolls sideways; `offender` names
the element poking past the right edge, which is the §8 overflow hunt from
`skills/doesntbreak/references/patterns.md` done automatically.

### Path B: npx Playwright fallback

Write a small script to the scratchpad directory and run it with Node. Confirm
Playwright is usable first (`npx playwright --version`); if the browser binary is
missing, run `npx playwright install chromium` once. The script loads the target at
each width, runs the same in-page measurement as Path A, saves `shot-<width>.png`,
and prints one JSON line per width. Use `waitUntil: 'networkidle'` on navigation.

## Output format

Report exactly these sections.

### Overflow results

A table, one row per width:

`width` | `PASS` / `FAIL` | offending element when FAIL (`tag#id.class`, measured right edge vs viewport)

PASS means no horizontal scroll (`overflow <= 1`). FAIL means the page scrolls
sideways at that width.

### Screenshots

Save the PNGs to a dedicated directory outside the repo (a temp or scratchpad path),
not the working tree, so a check run never dirties the user's git status. Name them
by width, for example `shot-320.png`. List the saved paths, one per width, so they
can be opened for before/after comparison. Note: the Playwright MCP may also drop a
`.playwright-mcp/` scratch folder in the working directory; do not commit it.

### Verdict

One line. Either "No horizontal scroll at 320 / 375 / 768 / 1440px." or name the
widths that FAIL and the dominant offending element. If this was run against a known
fixture pair, note that `examples/broken.html` is expected to FAIL and
`examples/fixed.html` is expected to PASS.

## Constraints

- Measure, do not fix. This command reports browser-observed facts and saves images.
  It must not edit the target or any source file.
- Every FAIL must carry a real measured offender from the page, not a guess.
- If the target needs a server you cannot reach, say so rather than reporting a
  misleading PASS.
