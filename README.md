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

### `qlcplus`

QLC+ (Q Light Controller Plus) lighting programming assistant for workspace files (`.qxw`), fixture definitions (`.qxf`), and show design/debugging workflows.

**What it does:**
- Helps with Scene, Chaser, Sequence, Collection, EFX, RGB Matrix, and Show Manager setup
- Explains and troubleshoots HTP/LTP conflicts, fade behavior, crossfades, and timing drift
- Guides Virtual Console design (Cue List, Solo Frame, sliders, button modes) and control strategy
- Assists with fixture modes/heads, channel mapping, input-output plugins (DMX USB, Art-Net, E1.31, MIDI, OSC), and malformed/corrupted `.qxw` fixes
- Provides practical timing checks and best-practice programming guidance for music-synced shows

### `code-review`

Reviews code changes (diffs, commits, branches, PRs) for bugs, structure issues, and unintentional behavior changes.

**What it does:**
- Automatically determines what to review based on input (uncommitted changes, commit hash, branch name, or PR)
- Reads full file context around changes to avoid false positives
- Focuses on bugs (logic errors, edge cases, security issues, broken error handling)
- Flags structural issues, obvious performance problems, and potentially unintentional behavior changes
- Provides actionable, severity-appropriate feedback without style nitpicking

### `flatpak-flathub`

Creates complete, Flathub-ready Flatpak packages — manifest, MetaInfo, desktop file, README, and PR description.

**What it does:**
- Guides the full packaging workflow from gathering app info to submitting a Flathub PR
- Generates correct manifests for multiple build systems (Meson, CMake, Cargo, npm/pnpm, pip, Go, .NET)
- Writes AppStream MetaInfo XML that passes `flatpak-builder-lint`
- Handles Electron apps (zypak, BaseApp), proprietary extra-data apps, and repackaging from Snap/AppImage
- Covers plugin/extension addons, offline dependency generation, and sandbox permission configuration
- Provides runtime selection guidance (GNOME 49, KDE 6.9, Freedesktop 25.08)

## Install

```bash
npx skills add razzeee/skills --skill commit
```

```bash
npx skills add razzeee/skills --skill qlcplus
```

```bash
npx skills add razzeee/skills --skill code-review
```

```bash
npx skills add razzeee/skills --skill flatpak-flathub
```

Or install all skills:

```bash
npx skills add razzeee/skills --all
```
