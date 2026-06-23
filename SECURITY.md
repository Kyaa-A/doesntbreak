# Security Policy

doesntbreak is a documentation-only Claude Agent Skill: it ships Markdown files (`SKILL.md`, `references/patterns.md`) with no executable code, no build step, and no runtime. The practical security surface is small — but skill content becomes instructions an AI agent will follow, so a few things are worth taking seriously.

## Reporting a vulnerability

Please report security concerns privately to **asnaripacalna@gmail.com** rather than opening a public issue. Include:

- a description of the issue and its impact,
- the file and lines involved, and
- steps to reproduce, if applicable.

Expect an initial response within a few days. Please allow a reasonable window to address the issue before any public disclosure.

## In scope

- Content in `SKILL.md` or `references/patterns.md` that could induce an agent to produce insecure code (for example, a CSS/HTML snippet carrying an injection or data-exfiltration vector).
- Misleading or hidden instructions that could cause unsafe behavior in a tool-using agent.

## Out of scope

- Vulnerabilities in Claude, Claude Code, or any host application that loads this skill — report those to their respective vendors.
- General responsive-design disagreements; those are regular issues or PRs.

## Supported versions

This project tracks a single line of development on `main`. Fixes land on `main`; there are no separately maintained release branches.
