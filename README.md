# razzeee/skills

A collection of agent skills for AI coding assistants.

## Skills

### `commit`

Creates a well-crafted git commit — inferring style, scope, and message quality from the repo's own history rather than defaulting to conventional commits.

**What it does:**
- Reads `git log` to match the repo's existing commit style (conventional commits, plain prose, ticket prefixes, etc.)
- Creates a feature branch automatically if you're on a protected branch (`main`, `master`, `develop`, `staging`, `release/*`)
- Selectively stages files — excludes debug code, `.env` files, scratch files, and anything unrelated to the work; tells you what it left out
- Splits into multiple commits when the staged changes are clearly unrelated
- Writes a body when the change warrants one (non-obvious bug fixes, refactors with tradeoffs, breaking changes)

## Install

```bash
npx skills add razzeee/skills --skill commit
```

Or install all skills:

```bash
npx skills add razzeee/skills --all
```
