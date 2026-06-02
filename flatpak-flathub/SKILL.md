---
name: flatpak-flathub
description: >
  Creates complete, Flathub-ready Flatpak packages — manifest, MetaInfo,
  desktop file, README, and PR description. Use whenever the user is packaging
  or submitting an app to Flatpak/Flathub: writing a manifest (app-id,
  finish-args, sdk-extensions, buildsystem), generating offline dependency
  sources (cargo-sources.json, generated-sources.json, python3-modules.json),
  writing AppStream metainfo.xml, setting sandbox permissions, fixing Flathub
  linter errors, choosing a runtime (GNOME 49 / KDE 6.9 / Freedesktop 25.08),
  or preparing a Flathub submission PR. Covers Rust/Cargo, Python/Meson,
  Node/Electron/pnpm, Go, .NET, GTK, Qt, CMake. Use for finish-args, OARS
  ratings, branding colors, flatpak-builder vs org.flatpak.Builder, Electron
  BaseApp/zypak, extra-data (Spotify, Slack, Zoom, VS Code), repackaging from
  Snap/AppImage/.deb for Flathub. Do not use for creating native
  snap/AppImage/deb/AUR packages, general Linux distro questions, or
  "flatpak vs X" comparisons.
---

# Flatpak / Flathub Packaging Skill

Your goal is to produce a complete, correct, submission-ready Flatpak package.
The output should build cleanly with `flatpak-builder`, pass the
`flatpak-builder-lint` linter, and meet all Flathub requirements so that a PR
can be opened against the `new-pr` branch of `github.com/flathub/flathub`.

---

## Workflow overview

1. **Gather information** — ask only what you cannot infer
2. **Choose a runtime** — see Runtime selection below
3. **Write the manifest** — `<AppID>.json` or `.yaml` at repo root
4. **Write MetaInfo** — `<AppID>.metainfo.xml` inside the app source
5. **Write the desktop file** — `<AppID>.desktop` inside the app source
6. **Add `flathub.json`** if architecture needs restricting
7. **Write `.gitignore`** — always include
8. **Write README.md** — build/lint/update instructions
9. **Explain next steps** — local build, lint, PR instructions

---

## Step 1 — Gather information

Ask (or infer from context) the following before writing anything:

| Question | Why it matters |
|---|---|
| Application name and what it does | Needed for MetaInfo `name`/`summary`/`description` |
| Application ID (reverse-DNS, e.g. `io.github.alice.MyApp`) | Central identifier; drives filenames and verification |
| Upstream source location (git URL + tag/commit, or archive URL + sha256) | `sources` block in the manifest |
| Build system (`meson`, `cmake`, `autotools`, `cargo`, `npm`, `pip`, …) | Determines `buildsystem` and config-opts |
| Runtime preference (`GNOME`, `KDE`, `Freedesktop`) | See Runtime selection |
| What the app needs at runtime (display, audio, network, files…) | Drives `finish-args` |
| Author/developer name and contact URL | MetaInfo `developer` block and `url` tags |
| License (SPDX identifier, e.g. `GPL-3.0-only`) | MetaInfo `project_license` |
| Whether it should build on both `x86_64` and `aarch64` | Whether `flathub.json` is needed |

If the user has already provided most of this (e.g. pasted a GitHub URL), extract
what you can and only ask about genuine gaps. Never ask for information you can
look up yourself.

---

## Step 2 — Application ID rules

The ID must follow reverse-DNS format `{tld}.{vendor}.{product}` and these rules:

- At least 3 components, no more than 5; max 255 characters total
- Characters `[A-Za-z0-9_]` in each component; a dash `-` only in the **last** component
- Domain portion in lowercase; convert `-` to `_`; prefix digit-starting components with `_`
- Must **not** end in `.desktop`, `.app`, `.linux`
- GitHub/GitLab/Codeberg hosted → use `io.github.`, `io.gitlab.`, `page.codeberg.` prefix with at least 4 components
- `org.gnome.`, `org.kde.`, `com.system76.` are protected — only official projects may use them

---

## Step 3 — Runtime selection

Pick the right runtime for the app's toolkit. Use the **current** stable versions below.

| Toolkit / stack | runtime | sdk | Current branch |
|---|---|---|---|
| GTK 4 / libadwaita / GNOME | `org.gnome.Platform` | `org.gnome.Sdk` | `49` |
| GTK 3 (GNOME) | `org.gnome.Platform` | `org.gnome.Sdk` | `49` |
| Qt 6 / KDE | `org.kde.Platform` | `org.kde.Sdk` | `6.9` |
| Qt 5 / KDE | `org.kde.Platform` | `org.kde.Sdk` | `5.15-24.08` |
| Electron / generic | `org.freedesktop.Platform` | `org.freedesktop.Sdk` | `25.08` |
| Python / CLI with GUI | `org.freedesktop.Platform` | `org.freedesktop.Sdk` | `25.08` |

**Never use an EOL runtime.** When uncertain, check
`https://docs.flathub.org/docs/for-app-authors/runtimes`.

---

## Step 4 — The manifest

The manifest must be named `<AppID>.json` (or `.yaml`) at the **top level** of
the repository.

### Minimal JSON template

```json
{
  "app-id": "io.github.alice.MyApp",
  "runtime": "org.gnome.Platform",
  "runtime-version": "49",
  "sdk": "org.gnome.Sdk",
  "command": "myapp",
  "finish-args": [
    "--share=ipc",
    "--socket=fallback-x11",
    "--socket=wayland",
    "--device=dri"
  ],
  "cleanup": [
    "/include",
    "/lib/pkgconfig",
    "*.la", "*.a"
  ],
  "modules": [
    {
      "name": "myapp",
      "buildsystem": "meson",
      "sources": [
        {
          "type": "git",
          "url": "https://github.com/alice/myapp.git",
          "tag": "v1.2.3",
          "commit": "<full sha>"
        }
      ]
    }
  ]
}
```

### Key manifest rules

- **All files stay in the packaging repo** — every `type: file` source path must
  be relative to the manifest file's directory. Never reference `../` or absolute
  paths outside the repo. The packaging repo is self-contained.
- **No network during build** — `--share=network` in `build-args` does nothing;
  all sources must be declared with public URLs
- Dependencies with package managers (cargo, npm, pip, go) need a pre-generated
  lockfile manifest; see `flatpak-builder-tools` at
  `https://github.com/flatpak/flatpak-builder-tools`
- Use `shared-modules` (git submodule) for common deps like `libayatana-appindicator`,
  `lua`, `SDL_sound`, etc.
- License files must be installed to `$FLATPAK_DEST/share/licenses/$FLATPAK_ID/`
- Put the **main app module last**; frequently-changing modules last within their
  group so rebuilds are faster
- Use `rename-icon`, `rename-desktop-file`, `rename-appdata-file` when upstream
  filenames don't already carry the app ID prefix

### finish-args quick reference

Only request what the app genuinely needs.

```
# Display
--share=ipc                     # Always pair with X11
--socket=fallback-x11           # X11 fallback (use with --socket=wayland)
--socket=wayland                # Native Wayland
--device=dri                    # GPU / OpenGL

# Audio
--socket=pulseaudio             # Sound I/O

# Network
--share=network                 # Internet access

# Filesystem (prefer portals over blanket access)
--filesystem=xdg-documents      # Documents folder
--filesystem=xdg-download       # Downloads folder
--filesystem=xdg-music:ro       # Music, read-only
--filesystem=home               # Whole home (avoid if possible)

# D-Bus (use specific names, never --socket=session-bus)
--talk-name=org.freedesktop.Notifications
--own-name=org.mpris.MediaPlayer2.myapp
```

Use portals (via `xdg-desktop-portal`) instead of blanket `--filesystem=home`
wherever possible. Avoid `--device=all` unless absolutely necessary.

---

## Step 5 — MetaInfo file

The MetaInfo file is written **in the packaging repo** alongside the manifest,
then installed to `/app/share/metainfo/<AppID>.metainfo.xml` at build time via
a `type: file` source entry. Never reference files outside the packaging repo
directory — all paths in `type: file` sources must be relative to the manifest.

It must pass `appstreamcli validate` (run via `flatpak-builder-lint appstream`).

### Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component type="desktop-application">
  <id>io.github.alice.MyApp</id>

  <metadata_license>CC0-1.0</metadata_license>
  <project_license>GPL-3.0-only</project_license>

  <name>My App</name>
  <summary>A short one-line description (no trailing period)</summary>

  <developer id="io.github.alice">
    <name>Alice</name>
  </developer>

  <description>
    <p>A paragraph describing what the app does.</p>
    <ul>
      <li>Feature one</li>
      <li>Feature two</li>
    </ul>
  </description>

  <launchable type="desktop-id">io.github.alice.MyApp.desktop</launchable>

  <branding>
    <color type="primary" scheme_preference="light">#4a90d9</color>
    <color type="primary" scheme_preference="dark">#2c5f8a</color>
  </branding>

  <!-- Pick brand colors that match the app's visual identity.
       Preview them at https://docs.flathub.org/banner-preview -->

  <content_rating type="oars-1.1" />

  <url type="homepage">https://github.com/alice/myapp</url>
  <url type="bugtracker">https://github.com/alice/myapp/issues</url>
  <url type="vcs-browser">https://github.com/alice/myapp</url>

  <screenshots>
    <screenshot type="default">
      <image>https://raw.githubusercontent.com/alice/myapp/main/screenshots/main.png</image>
      <caption>Main window</caption>
    </screenshot>
  </screenshots>

  <releases>
    <release version="1.2.3" date="2025-01-15">
      <description>
        <p>Initial Flathub release.</p>
      </description>
    </release>
  </releases>

  <requires>
    <control>keyboard</control>
    <control>pointing</control>
    <display_length compare="ge">768</display_length>
  </requires>
</component>
```

### MetaInfo checklist

- `<id>` must exactly match the Flatpak app-id
- `<metadata_license>` = license of the XML file itself (use `CC0-1.0`)
- `<project_license>` = SPDX identifier of the app; proprietary → `LicenseRef-proprietary=https://…`
- `<developer id="…">` requires an `id` attribute in reverse-DNS form
- Screenshot URLs must be direct image links, not branch URLs; use a tag/commit hash
- `<releases>` must be present and dates must not be in the future
- OARS: generate at `https://hughsie.github.io/oars/generate.html`; use `type="oars-1.1"`
- For unofficially packaged proprietary apps, add as first `<p>` in `<description>`:
  `<p>**This is a community package of APP NAME and not officially supported by UPSTREAM NAME.**</p>`

---

## Step 6 — Desktop file

The desktop file is written **in the packaging repo** alongside the manifest,
then installed via a `type: file` source entry — same pattern as MetaInfo.

```ini
[Desktop Entry]
Name=My App
Comment=A short one-line description
Exec=myapp
Icon=io.github.alice.MyApp
Terminal=false
Type=Application
Categories=Utility;
```

- Icon value = the AppID (no extension)
- Filename must be `<AppID>.desktop`
- `Categories` must use valid Menu Spec categories (not `GTK`, `Qt`, `GNOME`, etc.)

---

## Step 7 — flathub.json

Flathub builds on both `x86_64` and `aarch64` by default. Only include this
file if you need to **restrict** that — listing both arches is the same as
omitting the file entirely.

To build on one architecture only:

```json
{
  "only-arches": ["x86_64"]
}
```

To exclude one architecture:

```json
{
  "skip-arches": ["aarch64"]
}
```

If the app builds fine on both architectures, **do not create `flathub.json`**.

---

## Step 8 — Write .gitignore

Always create a `.gitignore` alongside the manifest:

```
.flatpak-builder
builddir
repo
```

These are the local build artefacts produced by `flatpak run org.flatpak.Builder` and should never be committed.

---

## Step 9 — Write README.md

Always create a `README.md` in the same directory as the manifest. Tailor the
content to whether the app is built from source or repackaged from a binary.

### Template for source-built apps

```markdown
# <AppID> Flatpak

Flatpak packaging for [App Name](https://upstream-url).

## Requirements

- [flatpak](https://flatpak.org/setup/)
- [org.flatpak.Builder](https://flathub.org/apps/org.flatpak.Builder) — always
  use the Flatpak-sandboxed builder, not a system `flatpak-builder`, to match
  the Flathub build environment

## Build and install locally

```bash
# Add the Flathub remote (needed for runtime dependencies)
flatpak remote-add --if-not-exists --user flathub \
  https://dl.flathub.org/repo/flathub.flatpakrepo

# Install the sandboxed builder, runtime, and SDK
flatpak install --user flathub org.flatpak.Builder
flatpak install --user flathub <runtime>/<arch>/<branch>
flatpak install --user flathub <sdk>/<arch>/<branch>

# Build and install
flatpak run org.flatpak.Builder --user --install --force-clean builddir <AppID>.json

# Run
flatpak run <AppID>
```

## Lint

```bash
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <AppID>.json
flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
```

## Update

To update to a new upstream version:
1. Update the `tag` and `commit` in the `sources` block of `<AppID>.json`
2. Update the `version` and `date` in the `<releases>` section of `<AppID>.metainfo.xml`
3. Run the linter and rebuild locally before submitting a PR
```

### Template for repackaged binary apps (proprietary / pre-built)

```markdown
# <AppID> Flatpak

Flatpak repackaging for [App Name](https://upstream-url) (proprietary).

> **Note:** This is a community-maintained package. It is not officially
> supported by the upstream developer.

## Requirements

- [flatpak](https://flatpak.org/setup/)
- [org.flatpak.Builder](https://flathub.org/apps/org.flatpak.Builder) — always
  use the Flatpak-sandboxed builder, not a system `flatpak-builder`, to match
  the Flathub build environment

## Build and install locally

```bash
# Add the Flathub remote
flatpak remote-add --if-not-exists --user flathub \
  https://dl.flathub.org/repo/flathub.flatpakrepo

# Install the sandboxed builder, runtime, and SDK
flatpak install --user flathub org.flatpak.Builder
flatpak install --user flathub org.freedesktop.Platform/<arch>/<branch>
flatpak install --user flathub org.freedesktop.Sdk/<arch>/<branch>

# Build and install
flatpak run org.flatpak.Builder --user --install --force-clean builddir <AppID>.json

# Run
flatpak run <AppID>
```

## Lint

```bash
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <AppID>.json
flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
```

## Update to a new upstream version

1. Find the new download URL from the upstream website
2. Download it and compute the new sha256:
   ```bash
   sha256sum <downloaded-file>
   ```
3. Update `url` and `sha256` in the `sources` block of `<AppID>.json`
4. Update the `version` and `date` in the `<releases>` section of `<AppID>.metainfo.xml`
5. Rebuild and lint locally before submitting a PR
```

Adapt the README to the specific app: fill in real runtime names, version
branches, and any app-specific notes (e.g. extra setup steps, known
limitations in the Flatpak sandbox, required D-Bus names, etc.).

---

## Step 9 — Explain next steps to the user

After producing the files, always explain:

1. **Build locally** to verify everything works:
   ```bash
   flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
   flatpak install --user flathub org.flatpak.Builder
   flatpak run org.flatpak.Builder --user --install --force-clean builddir <AppID>.json
   flatpak run <AppID>
   ```

2. **Run the linter**:
   ```bash
   flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <AppID>.json
   flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
   ```

3. **Submit to Flathub**:
   - Fork `github.com/flathub/flathub` with all branches
   - Clone and check out the `new-pr` branch
   - Create a new branch from `new-pr`, add files, push, open PR **against `new-pr`** (not `master`)
   - PR title: `Add <AppID>`
   - Fill in the PR description template — the PR body must address each checkbox:
     - A brief description of what the application does
     - A video showcasing the app running on Linux via Flatpak
     - Confirmation the App ID follows the naming rules
     - Confirmation you have read the submission requirements
     - Your relationship to the project (author / upstream contributor / third-party packager) — if third-party, include a link showing you contacted upstream

---

## Electron apps — runtime and launcher notes

**All** Electron Flatpaks — whether source-built or extra-data repackages — must
use `org.electronjs.Electron2.BaseApp` as the `base`. This provides zypak,
Chromium's setuid sandbox shim, and other Electron runtime essentials. The
`base-version` must match `runtime-version`.

```yaml
base: org.electronjs.Electron2.BaseApp
base-version: '25.08'
runtime: org.freedesktop.Platform
runtime-version: '25.08'
sdk: org.freedesktop.Sdk
separate-locales: false   # nearly always needed for Electron apps
```

### zypak launcher

zypak replaces Chromium's setuid sandbox with a Flatpak-compatible alternative.
Every Electron Flatpak needs a launcher script that calls `zypak-wrapper` (or
`zypak-wrapper.sh`). The key difference:

- **`zypak-wrapper.sh`** — use for source-built apps where the Electron binary
  is installed under `/app/main/` (e.g. from electron-builder output).
- **`zypak-wrapper`** (no `.sh`) — use for extra-data apps where the binary
  lands in `/app/extra/` at user install time.

Both forms require `TMPDIR` to be set to avoid Chromium crashes:

```bash
#!/bin/sh
export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
exec zypak-wrapper.sh /app/main/myapp "$@"
```

For extra-data apps the launcher is usually a `type: script` source:

```bash
export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
exec zypak-wrapper /app/extra/myapp/myapp "$@"
```

### patch-electron-desktop-filename

Electron apps that bundle their own `.asar` file sometimes hardcode the desktop
filename internally. Without patching, the app reports a wrong WM_CLASS to the
window manager, breaking taskbar grouping and notifications on Wayland/KDE.

`patch-electron-desktop-filename` is provided by `org.electronjs.Electron2.BaseApp`
and patches `resources/app.asar` in-place to report the correct AppID. Call it
as a build command **after** the binary is installed:

```yaml
build-commands:
  - install -Dm755 element.sh /app/bin/element
  - cp -r . /app/Element
  - patch-electron-desktop-filename ${FLATPAK_DEST}/Element/resources/app.asar
```

Known users: Element/Riot, Discord, Signal, Mattermost, GB Studio, Freelens.
Slack uses the older `patch-desktop-filename` variant (same purpose, different
command name — both are provided by the BaseApp).

You need this when:
- The app is a pre-built Electron binary (tarball or extra-data), not source-built
- The app uses Electron's desktop integration (notifications, taskbar, tray)
- The upstream `.asar` was not built with the Flatpak AppID as the desktop filename

### Sandbox notes

- Do **not** use `--no-sandbox` or `ELECTRON_DISABLE_SANDBOX=1` as a default.
  zypak handles the sandbox correctly for virtually all apps.
- `ZYPAK_DISABLE_SANDBOX=1` is only needed for specific apps (e.g. Discord)
  where zypak's sandbox causes crashes — treat it as a last resort and document
  why it's needed.
- `--allow=devel` in `finish-args` is sometimes needed for apps that use
  `ptrace` (e.g. VS Code's debugger). Only add it when actually required.

---

## Proprietary apps — extra-data pattern

Some apps cannot be legally bundled in the Flatpak itself (e.g. Spotify, Slack,
VS Code, Zoom). For these, use `type: extra-data` sources: the binary is
downloaded by the **user** at install time, not embedded in the Flatpak. The
build step only installs a small `apply_extra` script and the static assets
(icon, desktop file, metainfo); the heavy binary arrives later via
`flatpak install`.

### Key rules for extra-data

- Add `tags: [proprietary]` at the top of the manifest.
- The `extra-data` source requires `filename`, `url`, `sha256`, and `size`
  (in bytes). `size` must be exact — get it from `curl -sI <url> | grep -i content-length`.
- Each arch that needs a different binary gets its own `extra-data` source block
  with `only-arches: [x86_64]` / `only-arches: [aarch64]`.
- The `apply_extra` script is executed at install time inside a sandbox. It
  receives the downloaded file(s) in the working directory and must unpack them
  to the current dir. Install it to `/app/bin/apply_extra`.
- Extracted files land in `/app/extra/` at runtime. The launcher script
  typically calls `zypak-wrapper.sh /app/extra/<appname>/<binary>`.

### Extra-data manifest skeleton

```yaml
app-id: com.example.ProprietaryApp
runtime: org.freedesktop.Platform
runtime-version: '25.08'
sdk: org.freedesktop.Sdk
base: org.electronjs.Electron2.BaseApp   # omit if not Electron
base-version: '25.08'
command: myapp
tags: [proprietary]
separate-locales: false

modules:
  - name: myapp
    buildsystem: simple
    build-commands:
      - install -Dm755 apply_extra ${FLATPAK_DEST}/bin/apply_extra
      - install -Dm755 myapp.sh ${FLATPAK_DEST}/bin/myapp
      - install -Dm644 ${FLATPAK_ID}.metainfo.xml ${FLATPAK_DEST}/share/metainfo/${FLATPAK_ID}.metainfo.xml
      - install -Dm644 ${FLATPAK_ID}.desktop ${FLATPAK_DEST}/share/applications/${FLATPAK_ID}.desktop
      - install -Dm644 icon.png ${FLATPAK_DEST}/share/icons/hicolor/512x512/apps/${FLATPAK_ID}.png
    sources:
      - type: extra-data
        filename: myapp.tar.gz
        only-arches: [x86_64]
        url: https://example.com/releases/myapp-x86_64.tar.gz
        sha256: FULL_SHA256_HERE
        size: 123456789          # exact Content-Length in bytes
        x-checker-data:
          type: rotating-url
          url: https://example.com/releases/latest/myapp-x86_64.tar.gz
      - type: extra-data
        filename: myapp.tar.gz
        only-arches: [aarch64]
        url: https://example.com/releases/myapp-aarch64.tar.gz
        sha256: FULL_SHA256_HERE
        size: 123456789
      - type: script
        dest-filename: apply_extra
        commands:
          - tar xf myapp.tar.gz --no-same-owner
          - rm myapp.tar.gz
          - mv myapp-* myapp
      - type: script
        dest-filename: myapp.sh
        commands:
          - export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
          - exec zypak-wrapper.sh /app/extra/myapp/myapp "$@"
      - type: file
        path: ${FLATPAK_ID}.metainfo.xml
      - type: file
        path: ${FLATPAK_ID}.desktop
      - type: file
        path: icon.png
```

---

## Repackaging from Snap

When the vendor only publishes a Snap, extract it with `unsquashfs` inside
`apply_extra`. You need `squashfs-tools` available in the build environment —
include it from `shared-modules`:

```yaml
modules:
  - shared-modules/squashfs-tools/squashfs-tools.json
  - shared-modules/lzo/lzo.json    # squashfs-tools may need lzo

  - name: myapp
    buildsystem: simple
    build-commands:
      - install -Dm755 apply_extra ${FLATPAK_DEST}/bin/apply_extra
      - install -Dm755 myapp.sh ${FLATPAK_DEST}/bin/myapp
      - install -Dm644 ${FLATPAK_ID}.metainfo.xml ${FLATPAK_DEST}/share/metainfo/${FLATPAK_ID}.metainfo.xml
      - install -Dm644 ${FLATPAK_ID}.desktop ${FLATPAK_DEST}/share/applications/${FLATPAK_ID}.desktop
      - install -Dm644 icon.png ${FLATPAK_DEST}/share/icons/hicolor/512x512/apps/${FLATPAK_ID}.png
    sources:
      - type: extra-data
        filename: myapp.snap
        only-arches: [x86_64]
        url: https://api.snapcraft.io/api/v1/snaps/download/<SNAP_REVISION>.snap
        sha256: FULL_SHA256_HERE
        size: 123456789
        x-checker-data:
          type: snapcraft
          name: myapp
          channel: stable
          is-main-source: true
      - type: script
        dest-filename: apply_extra
        commands:
          - unsquashfs -quiet -no-progress myapp.snap usr/lib/myapp
          - mv squashfs-root/usr/lib/myapp/* .
          - rm -r squashfs-root myapp.snap
      - type: script
        dest-filename: myapp.sh
        commands:
          - export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
          - exec zypak-wrapper.sh /app/extra/myapp "$@"
```

**Getting the Snap URL and SHA256:**
```bash
# Find the snap revision URL
curl -s "https://api.snapcraft.io/v2/snaps/info/SNAPNAME" \
  -H "Snap-Device-Series: 16" | python3 -m json.tool | grep -A5 '"amd64"'
# Then fetch the .snap file and sha256sum it
```

**x-checker-data for snaps** uses `type: snapcraft` with `name` and `channel`.
The URL in the manifest is a specific revision URL (immutable); the checker
updates it on each new snap release.

---

## Repackaging from AppImage

AppImages are self-contained ELF executables with a squashfs payload. Extract
with `unsquashfs` just like snaps:

```yaml
modules:
  - shared-modules/squashfs-tools/squashfs-tools.json

  - name: myapp
    buildsystem: simple
    build-commands:
      - install -Dm755 apply_extra ${FLATPAK_DEST}/bin/apply_extra
      - install -Dm755 myapp.sh ${FLATPAK_DEST}/bin/myapp
      # ... metainfo, desktop, icon installs
    sources:
      - type: extra-data
        filename: myapp.AppImage
        only-arches: [x86_64]
        url: https://github.com/example/myapp/releases/download/v1.0/myapp-x86_64.AppImage
        sha256: FULL_SHA256_HERE
        size: 123456789
        x-checker-data:
          type: json
          url: https://api.github.com/repos/example/myapp/releases/latest
          version-query: .tag_name | ltrimstr("v")
          url-query: .assets[] | select(.name | endswith("x86_64.AppImage")) | .browser_download_url
          is-main-source: true
      - type: script
        dest-filename: apply_extra
        commands:
          # AppImage offset varies; use --offset or rely on magic byte skip
          - chmod +x myapp.AppImage
          - ./myapp.AppImage --appimage-extract
          - mv squashfs-root myapp
          - rm myapp.AppImage
      - type: script
        dest-filename: myapp.sh
        commands:
          - export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
          - exec zypak-wrapper.sh /app/extra/myapp/myapp "$@"
```

**Note:** AppImage extraction requires `--appimage-extract` flag (works on all
AppImage types) or direct `unsquashfs` with the right byte offset. The
`--appimage-extract` approach is more portable.

---

## Plugin / extension addons

Some apps expose an **extension point** that lets separately packaged plugins
install files into the host app's prefix. OBS Studio, LibreOffice macros, and
GIMP plug-ins are common examples.

### Parent app — declare the extension point

Add an `add-extensions` block to the parent manifest and create the target
directory in a post-install step:

```yaml
add-extensions:
  com.example.MyApp.Plugin:
    version: stable
    directory: plugins          # relative to /app
    add-ld-path: lib
    merge-dirs: lib/plugins;share/plugins
    subdirectories: true
    no-autodownload: true
    autodelete: false

modules:
  - name: myapp
    buildsystem: meson
    post-install:
      - install -d /app/plugins
    sources: [...]
```

The `merge-dirs` value must cover **every path** that plugin modules will
install files into, relative to the plugin's own prefix. Get these wrong and
the plugin files will not appear to the host app at runtime.

### Plugin addon manifest

```yaml
id: com.example.MyApp.Plugin.FooPlugin
branch: stable
runtime: com.example.MyApp          # the parent app IS the runtime
runtime-version: stable
sdk: org.kde.Sdk//6.9               # MUST match the SDK the parent was built with
build-extension: true               # tells flatpak-builder this is an extension
separate-locales: false

build-options:
  prefix: /app/plugins/FooPlugin    # must be inside the parent's extension directory

modules:
  - name: fooplugin
    buildsystem: cmake-ninja
    config-opts:
      - -DCMAKE_BUILD_TYPE=RelWithDebInfo
    post-install:
      - install -Dm644 LICENSE "${FLATPAK_DEST}/share/licenses/${FLATPAK_ID}/LICENSE"
    sources:
      - type: git
        url: https://github.com/example/fooplugin.git
        tag: v1.0.0
        commit: <full-sha>
```

**Critical rules for extension addons:**

- `runtime` = the parent app ID (not a freedesktop/gnome/kde runtime)
- `sdk` = the **same SDK** the parent app was built with (e.g. if the parent
  uses `org.kde.Sdk//6.9`, the extension must too — using `org.freedesktop.Sdk`
  will cause linker errors or ABI mismatches at runtime)
- `build-extension: true` is mandatory
- `prefix` must be a subdirectory of the parent's `directory` value (e.g.
  `/app/plugins/FooPlugin`)
- Do **not** add `finish-args` to an extension manifest; sandbox is inherited
  from the parent
- All dependencies that aren't in the parent's SDK must be bundled in the
  extension manifest as modules before the main plugin module

### Extension MetaInfo

Extensions use `type="addon"` and must declare what they extend:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component type="addon">
  <id>com.example.MyApp.Plugin.FooPlugin</id>
  <extends>com.example.MyApp</extends>

  <metadata_license>CC0-1.0</metadata_license>
  <project_license>GPL-2.0-only</project_license>

  <name>Foo Plugin</name>
  <summary>A short description of what the plugin does</summary>

  <developer id="io.github.alice">
    <name>Alice</name>
  </developer>

  <description>
    <p>Longer description of the plugin.</p>
  </description>

  <url type="homepage">https://github.com/alice/fooplugin</url>
  <url type="bugtracker">https://github.com/alice/fooplugin/issues</url>

  <content_rating type="oars-1.1" />

  <releases>
    <release version="1.0.0" date="2025-01-15">
      <description>
        <p>Initial release.</p>
      </description>
    </release>
  </releases>
</component>
```

Note: addon MetaInfo does **not** need `<launchable>`, `<screenshots>`, or
`<branding>` (those belong to the parent app).

---

## Language / framework notes

### Python

Python 3 is included in all current runtimes (GNOME, KDE, Freedesktop) — no
`sdk-extensions` entry is needed for pure Python apps.

#### Generating the Python dependency manifest

pip dependencies **cannot be fetched at build time** (no network). Use
`flatpak-pip-generator` from
[flatpak-builder-tools](https://github.com/flatpak/flatpak-builder-tools/tree/master/pip)
to pre-resolve and download all wheels/sdists into a lockfile-style JSON that
flatpak-builder can use as offline sources.

**Install the tool:**

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
cd flatpak-builder-tools/pip
pip install --user requirements-parser
```

**Generate from a `requirements.txt`:**

```bash
./flatpak-pip-generator \
  --runtime org.gnome.Sdk//49 \
  --requirements-file requirements.txt \
  --output python3-modules
# produces python3-modules.json
```

**Generate from a `pyproject.toml`:**

```bash
./flatpak-pip-generator \
  --runtime org.gnome.Sdk//49 \
  --pyproject-file pyproject.toml \
  --output python3-modules
```

**For packages with compiled extensions (need platform wheels for both arches):**

```bash
./flatpak-pip-generator \
  --runtime org.gnome.Sdk//49 \
  --prefer-wheels=grpcio,cryptography \
  --requirements-file requirements.txt \
  --output python3-modules
```

The `--runtime` flag must match the runtime used in the manifest (so pip runs
inside the correct Python version). Replace `org.gnome.Sdk//49` with
`org.kde.Sdk//6.9` or `org.freedesktop.Sdk//25.08` as appropriate.

**Include the generated file in the manifest:**

```json
"modules": [
  "python3-modules.json",
  {
    "name": "myapp",
    "buildsystem": "meson"
  }
]
```

or in YAML:

```yaml
modules:
  - python3-modules.json
  - name: myapp
    buildsystem: meson
```

**Regenerate whenever dependencies change.** Commit `python3-modules.json`
alongside the manifest — it is a reproducible lockfile, not a generated
artifact to be ignored.

Add the following to the README's **Update** section whenever the app uses
Python deps:

```markdown
### Updating Python dependencies

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
pip install --user requirements-parser
./flatpak-builder-tools/pip/flatpak-pip-generator \
  --runtime org.gnome.Sdk//49 \
  --requirements-file requirements.txt \
  --output python3-modules
```
```

### Rust / Cargo

**SDK extension:** Add `org.freedesktop.Sdk.Extension.rust-stable` to `sdk-extensions`. The
extension branch must match the runtime version (e.g. `//25.08` for Freedesktop 25.08; GNOME
and KDE runtimes are based on Freedesktop so the same branch is used).

**Generate `cargo-sources.json`:**

```bash
# Install flatpak-builder-tools (cargo generator)
git clone https://github.com/flatpak/flatpak-builder-tools.git
cd flatpak-builder-tools/cargo
pip install poetry
poetry install
poetry run python3 flatpak-cargo-generator.py /path/to/Cargo.lock -o cargo-sources.json
```

**Manifest (simple buildsystem):**

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.rust-stable

modules:
  - name: myapp
    buildsystem: simple
    build-options:
      append-path: /usr/lib/sdk/rust-stable/bin
      env:
        CARGO_HOME: /run/build/myapp/cargo
        CARGO_NET_OFFLINE: 'true'
    build-commands:
      - cargo --offline fetch --manifest-path Cargo.toml --verbose
      - cargo build --offline --release
      - install -Dm755 target/release/myapp ${FLATPAK_DEST}/bin/myapp
    sources:
      - type: git
        url: https://github.com/example/myapp
        tag: v1.0.0
        commit: FULL_COMMIT_SHA
      - cargo-sources.json
```

**Manifest (meson buildsystem — meson manages CARGO_HOME):**

If `meson.build` sets `CARGO_HOME` itself (e.g. `cargo-home` under the build root), cargo won't
find the generated config. Work around by copying it:

```yaml
    buildsystem: meson
    build-options:
      append-path: /usr/lib/sdk/rust-stable/bin
      env:
        CARGO_NET_OFFLINE: 'true'
    sources:
      - type: git
        ...
      - cargo-sources.json
      - type: shell
        commands:
          - mkdir -p .cargo
          - cp -vf cargo/config .cargo/config.toml
```

**Regenerate whenever `Cargo.lock` changes.** Commit `cargo-sources.json`.

---

### Node / npm / yarn / pnpm / Electron

**Install the generator:**

```bash
pipx install 'git+https://github.com/flatpak/flatpak-builder-tools.git#subdirectory=node'
```

**Generate sources:**

```bash
# npm
flatpak-node-generator npm package-lock.json -o generated-sources.json

# yarn
flatpak-node-generator yarn yarn.lock -o generated-sources.json

# pnpm
flatpak-node-generator pnpm pnpm-lock.yaml -o generated-sources.json
```

If the generated file exceeds GitHub's size limit, pass `-s` to split it into
`generated-sources.0.json`, `generated-sources.1.json`, etc.

**Node SDK extension** (needed if building Node from source or using native addons):
Add `org.freedesktop.Sdk.Extension.node22//25.08` (or `node24`) to `sdk-extensions` and
`append-path: /usr/lib/sdk/node22/bin`.

**Electron-specific notes:**
- Set `ELECTRON_CACHE` and `npm_config_cache` in `build-options.env` to point into
  `flatpak-node/` so Electron's prebuilt binaries are found offline.
- For ARM/ARM64 electron-builder, source
  `flatpak-node/electron-builder-arch-args.sh` to set `$ELECTRON_BUILDER_ARCH_ARGS`.
- Native addon ABI mismatches: pass `--electron-node-headers` to the generator and set
  `npm_config_nodedir` to `flatpak-node/node-gyp/electron-current`.
- Do **not** use `--socket=session-bus`; use specific `--talk-name=` instead.

**Regenerate whenever the lockfile changes.** Commit `generated-sources.json`.

---

### Go / Go Modules

**SDK extension:** Add `org.freedesktop.Sdk.Extension.golang` to `sdk-extensions`.

**Generate vendored sources:**

```bash
go install github.com/dennwc/flatpak-go-mod@latest
# Run inside the Go project directory (where go.mod lives):
flatpak-go-mod ./path/to/go/project
# Produces go.mod.yaml and modules.txt
```

**Manifest:**

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.golang

modules:
  - name: myapp
    buildsystem: simple
    build-options:
      append-path: /usr/lib/sdk/golang/bin
    build-commands:
      - go install -mod=vendor ./cmd/myapp
    sources:
      - type: git
        url: https://github.com/example/myapp
        commit: FULL_COMMIT_SHA
      - type: file
        path: modules.txt
        dest: vendor
      - go.mod.yaml   # include the generated sources
```

**Regenerate whenever `go.mod` / `go.sum` changes.** Commit both `go.mod.yaml` and `modules.txt`.

---

### .NET / NuGet

**SDK extension:** Add `org.freedesktop.Sdk.Extension.dotnet9` (or `dotnet8`) to
`sdk-extensions`. Use the branch matching the Freedesktop SDK version.

**Generate NuGet sources:**

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
cd flatpak-builder-tools/dotnet
# Requires org.freedesktop.Sdk + org.freedesktop.Sdk.Extension.dotnet installed locally
python3 flatpak-dotnet-generator.py nuget-sources.json MyApp.csproj \
  --runtime linux-x64 --freedesktop 25.08
```

**Manifest:**

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.dotnet9

modules:
  - name: myapp
    buildsystem: simple
    build-options:
      append-path: /usr/lib/sdk/dotnet9/bin
      env:
        DOTNET_CLI_TELEMETRY_OPTOUT: 'true'
        DOTNET_NOLOGO: 'true'
    build-commands:
      - '. /usr/lib/sdk/dotnet9/enable.sh'
      - dotnet publish -c Release --source nuget-sources -o /app/lib/myapp MyApp.csproj
      - install -Dm755 /dev/stdin ${FLATPAK_DEST}/bin/myapp <<'EOF'
        #!/bin/sh
        exec /app/lib/myapp/myapp "$@"
        EOF
    sources:
      - type: git
        url: https://github.com/example/myapp
        tag: v1.0.0
        commit: FULL_COMMIT_SHA
      - nuget-sources.json
```

Always pass `--source nuget-sources` to `dotnet build`/`dotnet publish` so it picks up
the offline sources. **Regenerate whenever `.csproj` or `packages.lock.json` changes.**

---

### Meson apps
- `buildsystem: meson` handles `meson setup / ninja / ninja install` automatically
- Pass options via `config-opts`, e.g. `"-Dtests=false"`

### CMake apps
- Use `buildsystem: cmake-ninja`
- `-DCMAKE_BUILD_TYPE=RelWithDebInfo` is a good default

---

## Linter errors quick guide

| Error | Fix |
|---|---|
| `appstream-metainfo-missing` | Install metainfo to `/app/share/metainfo/<AppID>.metainfo.xml` |
| `finish-args-contains-both-x11-and-fallback-x11` | Remove `--socket=x11`; keep only `--socket=fallback-x11` + `--socket=wayland` |
| bare `--socket=x11` without `fallback-x11` | Always prefer `--socket=fallback-x11` + `--socket=wayland`; bare `--socket=x11` is a linter warning unless the app truly cannot support Wayland |
| license not installed | Every module must install its LICENSE file: `install -Dm644 LICENSE "${FLATPAK_DEST}/share/licenses/${FLATPAK_ID}/LICENSE"` |
| `finish-args-arbitrary-dbus-access` | Replace `--socket=session-bus` with specific `--talk-name=` / `--own-name=` |
| `finish-args-has-nodevices` | Don't use `--device=all` when a narrower permission exists |
| `module-buildsystem-is-plain-cmake` | Use `cmake-ninja` instead of `cmake` |
| `desktop-file-icon-not-installed` | Icon file must be installed to `/app/share/icons/hicolor/<size>/apps/<AppID>.png` |

For a full explanation of any linter error: `https://docs.flathub.org/docs/for-app-authors/linter`

---

## Reference docs

- Flathub requirements: `https://docs.flathub.org/docs/for-app-authors/requirements`
- MetaInfo guidelines: `https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines`
- Flatpak manifest reference: `https://docs.flatpak.org/en/latest/manifests.html`
- Sandbox permissions: `https://docs.flatpak.org/en/latest/sandbox-permissions.html`
- Flatpak Builder Tools: `https://github.com/flatpak/flatpak-builder-tools`
- Shared modules: `https://github.com/flathub/shared-modules`
- AppStream MetaInfo Creator: `https://www.freedesktop.org/software/appstream/metainfocreator/`
- OARS generator: `https://hughsie.github.io/oars/generate.html`
- Existing Flathub manifests for reference: `https://github.com/flathub`
