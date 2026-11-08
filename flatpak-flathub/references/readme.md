# Packaging README templates

Choose the template for the app's distribution method. Fill in the actual app
ID, runtime, SDK, architecture, branch, and manifest filename. Include any
app-specific setup, D-Bus requirements, or sandbox limitations.

## Source-built apps

````markdown
# <AppID> Flatpak

Flatpak packaging for [App Name](https://upstream-url).

## Requirements

- [flatpak](https://flatpak.org/setup/)
- [org.flatpak.Builder](https://flathub.org/apps/org.flatpak.Builder). Use the
  Flatpak-sandboxed builder to match the Flathub build environment.

## Build and install locally

```bash
# Add the Flathub remote for runtime dependencies
flatpak remote-add --if-not-exists --user flathub \
  https://dl.flathub.org/repo/flathub.flatpakrepo

# Install the sandboxed builder, runtime, and SDK
flatpak install --user flathub org.flatpak.Builder
flatpak install --user flathub <runtime>/<arch>/<branch>
flatpak install --user flathub <sdk>/<arch>/<branch>

# Build, export the repository for linting, and install
flatpak run org.flatpak.Builder --user --install --repo=repo --force-clean builddir <AppID>.json

# Run
flatpak run <AppID>
```

## Lint

```bash
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <AppID>.json
flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
```

## Update

1. Update the `tag` and `commit` in the manifest's `sources` block.
2. Update packaged release metadata if it is maintained here rather than upstream.
3. Regenerate offline dependency sources when lockfiles change.
4. Rebuild and lint locally before submitting a PR.
````

## Repackaged binary apps

````markdown
# <AppID> Flatpak

Flatpak repackaging for [App Name](https://upstream-url) (proprietary).

> **Note:** This is a community-maintained package. It is not officially
> supported by the upstream developer.

## Requirements

- [flatpak](https://flatpak.org/setup/)
- [org.flatpak.Builder](https://flathub.org/apps/org.flatpak.Builder). Use the
  Flatpak-sandboxed builder to match the Flathub build environment.

## Build and install locally

```bash
flatpak remote-add --if-not-exists --user flathub \
  https://dl.flathub.org/repo/flathub.flatpakrepo

flatpak install --user flathub org.flatpak.Builder
flatpak install --user flathub org.freedesktop.Platform/<arch>/<branch>
flatpak install --user flathub org.freedesktop.Sdk/<arch>/<branch>

flatpak run org.flatpak.Builder --user --install --repo=repo --force-clean builddir <AppID>.json
flatpak run <AppID>
```

## Lint

```bash
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <AppID>.json
flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
```

## Update to a new upstream version

1. Find the new download URL from upstream.
2. Download the artifact and compute its SHA256 and size in bytes:
   ```bash
   sha256sum downloaded-file
   wc -c < downloaded-file
   ```
3. Update `url`, `sha256`, and (for extra-data) `size` in the manifest.
4. Update packaged release metadata when maintained here.
5. Rebuild, lint, install, and test locally before submitting a PR.
````

For Python, Rust, Node, Go, or .NET dependencies, include the regeneration
commands from the applicable language reference in the README's Update section.
