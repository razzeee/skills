# razzeee/skills

Agent skills for AI coding assistants.

## Skills

### `commit`

Creates commits that match the repository's existing style instead of imposing conventional commits.

**What it does**
- Reads `git log` to identify the repository's commit style, including prose, prefixes, and ticket numbers
- Creates a feature branch if the current branch is protected, such as `main`, `master`, `develop`, `staging`, or `release/*`
- Stages only files related to the change and reports anything it leaves out
- Splits into multiple commits when the staged changes are clearly unrelated
- Writes a body when the change warrants one (non-obvious bug fixes, refactors with tradeoffs, breaking changes)

### `qlcplus`

Helps program QLC+ shows, edit workspace files (`.qxw`), and create fixture definitions (`.qxf`).

**What it does**
- Helps with Scene, Chaser, Sequence, Collection, EFX, RGB Matrix, and Show Manager setup
- Explains and troubleshoots HTP/LTP conflicts, fade behavior, crossfades, and timing drift
- Guides Virtual Console setup for cue lists, solo frames, sliders, and button modes
- Helps configure fixtures, map channels, set up input and output plugins, and repair malformed `.qxw` files
- Checks timing for music-synced shows

### `code-review`

Reviews code changes (diffs, commits, branches, PRs) for bugs, structure issues, and unintentional behavior changes.

**What it does**
- Determines the review target from the input: uncommitted changes, a commit, a branch, or a pull request
- Reads full file context around changes to avoid false positives
- Focuses on bugs (logic errors, edge cases, security issues, broken error handling)
- Flags structural issues, obvious performance problems, and potentially unintentional behavior changes
- Reports actionable findings by severity without style nitpicking

### `flatpak-flathub`

Creates Flathub-ready Flatpak manifests, MetaInfo, desktop files, READMEs, and pull request descriptions.

**What it does**
- Guides the full packaging workflow from gathering app info to submitting a Flathub PR
- Generates correct manifests for multiple build systems (Meson, CMake, Cargo, npm/pnpm, pip, Go, .NET)
- Writes AppStream MetaInfo XML that passes `flatpak-builder-lint`
- Handles Electron apps (zypak, BaseApp), proprietary extra-data apps, and repackaging from Snap/AppImage
- Covers plugin/extension addons, offline dependency generation, and sandbox permission configuration
- Selects a current GNOME, KDE, or Freedesktop runtime

### `flatpak-sdk-extension-maintenance`

Propagates dependency and release updates across versioned Flatpak SDK and SDK-extension branches.

**What it does**
- Inventories every maintained `branch/*` runtime branch and classifies it as current, behind, divergent, or already covered by a pull request
- Builds a verified chain of original update commits for branches that are multiple releases behind
- Prepares updates in isolated worktrees while preserving branch-specific runtime settings
- Validates source checksums, manifests, release metadata, diff scope, and remote branch state
- Requires explicit approval before pushing update branches or opening pull requests

### `flatpak-c-conventions`

Applies Flatpak's C integer type conventions when writing, changing, or reviewing code in the Flatpak repository.

**What it does**
- Uses standard C integer types such as `size_t`, `uint8_t`, and `uint32_t` when appropriate
- Uses `size_t` for array indexes, collection lengths, and related loop counters
- Preserves GLib types where an API, signedness, width, or public interface requires them

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

```bash
npx skills add razzeee/skills --skill flatpak-sdk-extension-maintenance
```

```bash
npx skills add razzeee/skills --skill flatpak-c-conventions
```

Or install all skills:

```bash
npx skills add razzeee/skills --all
```
