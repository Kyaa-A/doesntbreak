---
description: Apply the doesntbreak responsive fixes to layout code. Finds the same offenders /doesntbreak:audit reports, edits the source to fix the high-confidence ones, flags the judgment calls, and verifies with a real browser. Leaves changes unstaged for review, never commits.
argument-hint: [path, glob, or component to fix, defaults to the project's UI files]
---

You are running the **apply** half of doesntbreak. Your job is to fix responsive
layout bugs in the source, then prove the fix worked. This is the write-mode
counterpart to `/doesntbreak:audit` (which only reports). You edit files, but you
leave every change unstaged for the user to review. You MUST NOT commit, stage, or
reformat anything beyond the targeted fixes.

## Rules source

Apply the `doesntbreak` rules as your rubric. Read `skills/doesntbreak/SKILL.md` for
the rules, and read `skills/doesntbreak/references/patterns.md` for the exact
corrected snippet for each fix. Use the canonical fix from `patterns.md`, not one
reasoned from memory.

## Scope

Target: `$ARGUMENTS`

- If an argument is given, scope to that path / glob / component. Edit only files
  inside that scope.
- If no argument is given, discover the project's UI layout files: `.html`, `.css`,
  `.scss`, `.jsx`, `.tsx`, `.vue`, `.svelte`, and class-bearing markup.
- If no UI/layout files are found, say so plainly and stop. Do not invent fixes.

## Procedure

1. **Baseline.** If the target renders, run the `/doesntbreak:screenshot-check` browser
   pass on the unedited target first and keep the per-width overflow numbers as the
   "before". If no browser is available, note that and continue without a baseline.
2. **Scan.** Run the 9-rule audit rubric to locate offenders with real `file:line`.
3. **Partition** each offender into auto-fix or flag (see the two lists below).
4. **Apply** each auto-fix in place with a minimal edit using the `patterns.md`
   snippet, matching the surrounding code style. Skip anything already compliant
   (idempotent). If a construct cannot be located safely (a framework abstraction
   hides the real element), flag it instead of forcing a change.
5. **Verify.** Re-run `/doesntbreak:screenshot-check` on the edited target and capture
   the "after" numbers.
6. **Leave for review.** Do not `git add`, do not commit, do not touch unrelated code.

## Auto-fix (one canonical fix each)

Apply these without asking:

1. **Fixed width** on a container: `width: 1200px` becomes `width: min(100% - 2rem, 1200px); margin-inline: auto` (Tailwind `w-[1200px]` becomes `w-full max-w-[1200px]`).
3. **Fixed height** on content: `height: Npx` / `height: 100vh` becomes `min-height: ...`.
4. **Unconstrained media**, when a global stylesheet or `<style>` block exists: add `img, video, svg { max-width: 100%; height: auto; }` there.
5. **Tap target** below 44px: add padding / `min-width: 44px; min-height: 44px`.
6. **Input font** below 16px: set `font-size: 16px` on the input.
7. **Text overflow**: add `min-width: 0` to the flex/grid child and `overflow-wrap: break-word` to the text node; grid `1fr` becomes `minmax(0, 1fr)`.
8. **Viewport height**: `100vh` on a full-height section becomes `min-height: 100vh; min-height: 100dvh` (fallback first, then `dvh`). Tailwind `min-h-screen` becomes `min-h-dvh`.
9. **Wide content** (table, `<pre>`, code block) with no scroll wrapper: wrap it in a container with `overflow-x: auto`.

## Flag, do not guess

Report these for the user to decide instead of auto-applying:

- **Rule 2, reflow.** A non-wrapping flex row has two valid fixes: `flex-wrap: wrap`
  (keeps the row intent) or an auto-fit grid
  (`grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr))`). Present
  both and let the user choose.
- **Rule 4, media**, when there is no obvious global stylesheet to hold the reset (a
  lone component with no shared CSS). Name the file and the two placement options.
- **Anything you cannot locate or fix safely.** A guessed edit is worse than a flag.

## Output format

### Applied fixes
A table, one row per edit: `file:line` | rule | before becomes after.

### Needs your decision
The flagged items, each with its 2 valid options. Omit this section if empty.

### Verification
Per width, before becomes after, for example `320: FAIL (overflow 1696) becomes PASS (-15)`.
End with a one-line verdict. If any width still FAILs, say so and name the likely
cause (a flagged item not yet applied, or a cause outside this file).

### Review note
Remind the user the changes are unstaged: review with `git diff`, then commit when
satisfied. Nothing was committed or staged.

## Constraints

- Edit only files in scope. Never commit, never stage, never reformat unrelated code.
- One rule is one minimal edit. Do not refactor surrounding code.
- Flag rather than force when uncertain.
- Every applied fix must trace to a real offender you found, with the canonical
  `patterns.md` fix. Do not invent changes.
