# Proprietary apps and binary repackaging

Check the upstream license before deciding whether binaries can be bundled.
When redistribution is not permitted, use `type: extra-data`: users download
the binary at install time. The build installs `apply_extra`, launchers, and
static metadata; the binary arrives during `flatpak install`.

## extra-data rules

- Add `tags: [proprietary]` for proprietary applications.
- Supply `filename`, `url`, `sha256`, and exact byte `size` for each extra-data
  source. Download the artifact and use `sha256sum` and `wc -c` to verify it.
- Use `only-arches: [x86_64]` / `only-arches: [aarch64]` for distinct binaries.
- Install `apply_extra` to `/app/bin/apply_extra`. It executes in an install-time
  sandbox with the downloads in its working directory, and extracts there.
- Extracted files appear under `/app/extra/` at runtime. Match the launcher to
  the extracted directory structure.
- For Electron, also read the Electron reference linked from `SKILL.md` and
  verify the selected BaseApp's wrapper command.

## Manifest skeleton

Replace the example branches with supported matching branches. Omit the
Electron BaseApp and zypak launcher for non-Electron apps.

```yaml
app-id: com.example.ProprietaryApp
runtime: org.freedesktop.Platform
runtime-version: '25.08'
sdk: org.freedesktop.Sdk
base: org.electronjs.Electron2.BaseApp
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
        size: 123456789
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
          - exec zypak-wrapper /app/extra/myapp/myapp "$@"
      - type: file
        path: com.example.ProprietaryApp.metainfo.xml
      - type: file
        path: com.example.ProprietaryApp.desktop
      - type: file
        path: icon.png
```

Use literal repository filenames in `path` fields; shell environment-variable
expansion in `build-commands` does not imply expansion in source paths.

## Repackaging from Snap

When upstream only publishes a Snap, extract it with `unsquashfs` in
`apply_extra`. Include the required extraction tools from `shared-modules`:

```yaml
modules:
  - shared-modules/squashfs-tools/squashfs-tools.json
  - shared-modules/lzo/lzo.json

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
          - exec zypak-wrapper /app/extra/myapp "$@"
```

Find the revision URL and metadata, then download the artifact to verify the
checksum and size:

```bash
curl -s "https://api.snapcraft.io/v2/snaps/info/SNAPNAME" \
  -H "Snap-Device-Series: 16" | python3 -m json.tool
```

Use an immutable revision URL in the manifest. `x-checker-data` with
`type: snapcraft`, `name`, and `channel` discovers new releases.

## Repackaging from AppImage

Determine the AppImage format and extraction requirements. Type 2 AppImages
contain a SquashFS payload; their runtime commonly supports
`--appimage-extract`. Extraction support is format-dependent, so do not assume
one command works for every AppImage. Direct `unsquashfs` extraction needs the
correct byte offset and the extraction tool available in the sandbox.

Example for an AppImage supporting `--appimage-extract`:

```yaml
modules:
  - name: myapp
    buildsystem: simple
    build-commands:
      - install -Dm755 apply_extra ${FLATPAK_DEST}/bin/apply_extra
      - install -Dm755 myapp.sh ${FLATPAK_DEST}/bin/myapp
      # Also install metainfo, desktop file, and icons.
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
          - chmod +x myapp.AppImage
          - ./myapp.AppImage --appimage-extract
          - mv squashfs-root myapp
          - rm myapp.AppImage
      - type: script
        dest-filename: myapp.sh
        commands:
          - export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
          - exec zypak-wrapper /app/extra/myapp/myapp "$@"
```
