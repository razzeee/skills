---
name: flatpak-flathub
metadata:
  author: razzeee
  source: https://github.com/razzeee/skills/tree/main/flatpak-flathub
description: >
  Creates Flathub-ready Flatpak manifests, MetaInfo, desktop files, READMEs,
  and pull request descriptions. Use when packaging or submitting an app to
  Flatpak/Flathub, configuring finish-args or SDK extensions, generating
  offline dependency sources, fixing Flathub linter errors, choosing a
  runtime, or preparing a submission PR. Covers Rust/Cargo, Python/Meson,
  Node/Electron/npm/yarn/pnpm, Go, .NET, GTK, Qt, and CMake; AppStream,
  OARS, branding, Electron BaseApp/zypak, proprietary extra-data, plugin
  addons, and repackaging Snap/AppImage/.deb files for Flathub. Do not use
  for creating native snap/AppImage/deb/AUR packages, general Linux distro
  questions, or comparisons of Flatpak with other packaging systems.
compatibility: Requires network access to verify upstream sources and current Flathub requirements. Local build and lint validation requires Linux, flatpak, and org.flatpak.Builder.
---

# Flatpak / Flathub Packaging

Produce a complete, submission-ready Flatpak package. Build with the
Flatpak-sandboxed `org.flatpak.Builder`, run its linter, and report which checks
passed or could not run. New application submissions target the `new-pr` branch
of `github.com/flathub/flathub`.

## Workflow

1. Gather the application details and inspect existing packaging.
2. Verify the app ID and choose a supported runtime.
3. Write the manifest and generate offline dependency sources.
4. Supply MetaInfo, a desktop file, and icons.
5. Add architecture restrictions only when needed, plus build documentation.
6. Build, lint, and test the application.
7. Explain the results and prepare the requested submission materials.

## 1. Gather information

Infer these from the repository and upstream sources; ask only about gaps:

| Information | Purpose |
|---|---|
| Application name and what it does | MetaInfo name, summary, description |
| Reverse-DNS application ID | Filenames, installation paths, verification |
| Upstream URL and tag/commit or archive checksum | Manifest sources |
| Build system and dependency lockfiles | Build commands and offline sources |
| Toolkit and runtime requirements | Runtime and SDK selection |
| Display, audio, network, filesystem needs | Sandbox permissions |
| Developer name, contact URL, SPDX license | MetaInfo |
| Supported architectures | Whether `flathub.json` is needed |

Check whether upstream already supplies suitable MetaInfo, desktop files, and
icons. Reuse them when possible. When supplying missing files in the packaging
repository, install them via repository-relative `type: file` sources.

## 2. Application ID and runtime

The application ID follows reverse-DNS format `{tld}.{vendor}.{product}`:

- At least 3 components, no more than 5; max 255 characters total.
- Use `[A-Za-z0-9_]` in each component; a dash is allowed only in the last component.
- Use lowercase for the domain; convert dashes to underscores and prefix digit-starting components with `_`.
- Do not end the ID in `.desktop`, `.app`, or `.linux`.
- GitHub/GitLab/Codeberg projects use `io.github.`, `io.gitlab.`, or `page.codeberg.` with at least 4 components.
- Protected namespaces such as `org.gnome.`, `org.kde.`, and `com.system76.` are for official projects.

Choose the runtime for the toolkit:

| Toolkit / stack | Runtime | SDK |
|---|---|---|
| GTK / libadwaita / GNOME | `org.gnome.Platform` | `org.gnome.Sdk` |
| Qt / KDE | `org.kde.Platform` | `org.kde.Sdk` |
| Electron / generic / Python without GNOME or KDE dependencies | `org.freedesktop.Platform` | `org.freedesktop.Sdk` |

Verify supported branches against the [Flathub runtime documentation](https://docs.flathub.org/docs/for-app-authors/runtimes)
and available runtime refs before selecting one. Match the app's toolkit version,
SDK extensions, and BaseApp to that runtime. GNOME and KDE version numbers differ
from their underlying Freedesktop SDK branch. Use a supported branch, not an EOL
one. If current support cannot be verified, report that gap.

Version numbers in the reference templates are examples, not a list of current
releases. Replace them consistently after checking support and compatibility.

## 3. Write the manifest

Read [manifest rules and template](references/manifest.md) when creating or
editing the manifest, sandbox permissions, or common dependency modules.

Name the manifest `<AppID>.json` or `<AppID>.yaml` at the repository root.
Keep packaging files self-contained. Declare all downloads as sources because
build steps have no network access. Put the main app module last.

Read each applicable reference before writing stack-specific build commands:

| Task | Reference |
|---|---|
| Python dependencies and pip source generation | [Python](references/python.md) |
| Rust/Cargo sources, including Meson-managed Cargo | [Rust](references/rust.md) |
| npm, yarn, pnpm, native Node addons, Electron sources | [Node](references/node.md) |
| Go modules and vendoring | [Go](references/go.md) |
| .NET/NuGet offline sources | [.NET](references/dotnet.md) |
| Any Electron app, source-built or repackaged | [Electron runtime and launchers](references/electron.md) |
| Proprietary extra-data, Snap, or AppImage repackaging | [Binary repackaging](references/repackaging.md) |
| Parent extension points and plugin addons | [Extensions](references/extensions.md) |

For Meson, `buildsystem: meson` handles setup, build, and install. Pass options
through `config-opts`, such as `"-Dtests=false"`. For CMake, use
`buildsystem: cmake-ninja`; `-DCMAKE_BUILD_TYPE=RelWithDebInfo` is a useful default.

## 4. MetaInfo and desktop integration

Read [MetaInfo and desktop files](references/desktop-integration.md) when writing
or validating application metadata, OARS ratings, branding, screenshots, or
desktop entries. For addon metadata, use the extensions reference above.

Install MetaInfo to `/app/share/metainfo/<AppID>.metainfo.xml`, the desktop file
to `/app/share/applications/<AppID>.desktop`, and icons under
`/app/share/icons/hicolor/<size>/apps/`. Ensure all identifiers agree.

## 5. Supporting files

### Architecture restrictions

Flathub builds on both `x86_64` and `aarch64` by default. Create `flathub.json`
only when restricting architectures. For example:

```json
{
  "only-arches": ["x86_64"]
}
```

Alternatively, `"skip-arches": ["aarch64"]` excludes that architecture.
Listing both architectures is equivalent to omitting the file.

### Build artifacts and README

Include these local build artifacts in `.gitignore`:

```gitignore
.flatpak-builder
builddir
repo
```

Write a README with build, lint, and update instructions using the applicable
[README template](references/readme.md). Fill in real runtime names, branches,
app-specific setup, sandbox limitations, and dependency regeneration commands.

## 6. Validate

Use the sandboxed builder to match Flathub's environment:

```bash
flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install --user flathub org.flatpak.Builder
flatpak run org.flatpak.Builder --user --install --install-deps-from=flathub --repo=repo --force-clean builddir <AppID>.json
flatpak run <AppID>
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <AppID>.json
flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
```

The build exports to `repo` so the repository lint command has an actual target.
Use the manifest's real filename, including `.yaml` when appropriate.

### Linter errors quick guide

| Error | Fix |
|---|---|
| `appstream-metainfo-missing` | Install MetaInfo to `/app/share/metainfo/<AppID>.metainfo.xml` |
| `finish-args-contains-both-x11-and-fallback-x11` | Keep only `--socket=fallback-x11` plus `--socket=wayland` |
| Bare `--socket=x11` | Prefer fallback-x11 plus Wayland when the app supports Wayland |
| License not installed | Install the module's license under `${FLATPAK_DEST}/share/licenses/${FLATPAK_ID}/` |
| `finish-args-arbitrary-dbus-access` | Replace session-bus access with specific `--talk-name=` / `--own-name=` |
| `finish-args-has-nodevices` | Check requested device permissions; prefer narrower access over `--device=all` |
| `module-buildsystem-is-plain-cmake` | Use `cmake-ninja` |
| `desktop-file-icon-not-installed` | Install an AppID-named icon in the appropriate hicolor directory |

Look up unfamiliar or changed diagnostics in the
[linter documentation](https://docs.flathub.org/docs/for-app-authors/linter).

## 7. Explain next steps

Report the files produced, build/lint results, runtime verification, and any
remaining blockers. Provide commands for checks that could not run locally.

For a new application submission:

1. Read the current Flathub submission requirements and PR template.
2. Fork `github.com/flathub/flathub` with the `new-pr` branch available.
3. Create the submission branch from `new-pr` and target `new-pr` in the PR.
4. Use the title `Add <AppID>` and fill in the current template with verified
   details: what the app does, demonstration material requested by the template,
   app ID compliance, and the submitter's relationship to upstream. Include
   evidence of upstream contact for third-party packaging where required.

Prepare submission materials as requested; publish only with the user's
authorization. Do not claim a build, runtime test, or checklist item passed
unless it was verified.

## Upstream references

- [Flathub requirements](https://docs.flathub.org/docs/for-app-authors/requirements)
- [MetaInfo guidelines](https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines)
- [Flatpak manifests](https://docs.flatpak.org/en/latest/manifests.html)
- [Sandbox permissions](https://docs.flatpak.org/en/latest/sandbox-permissions.html)
- [Flatpak Builder Tools](https://github.com/flatpak/flatpak-builder-tools)
- [Shared modules](https://github.com/flathub/shared-modules)
- [AppStream MetaInfo Creator](https://www.freedesktop.org/software/appstream/metainfocreator/)
- [OARS generator](https://hughsie.github.io/oars/generate.html)
- [Existing Flathub manifests](https://github.com/flathub)
