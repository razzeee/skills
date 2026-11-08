# Electron runtime and launchers

Use `org.electronjs.Electron2.BaseApp` for source-built and repackaged Electron
apps. It provides zypak and other Electron runtime essentials. Match the
BaseApp branch to the chosen Freedesktop runtime. These version numbers are
examples; verify supported branches before using them.

```yaml
base: org.electronjs.Electron2.BaseApp
base-version: '25.08'
runtime: org.freedesktop.Platform
runtime-version: '25.08'
sdk: org.freedesktop.Sdk
separate-locales: false
```

## zypak launcher

zypak provides a Flatpak-compatible alternative to Chromium's sandbox setup.
Use a launcher that calls the wrapper provided by the selected BaseApp.
Check its wrapper entry points and existing manifests for the same packaging
layout rather than assuming a `.sh` suffix from the download format.

For a source-built app installed under `/app/main/`:

```sh
#!/bin/sh
export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
exec zypak-wrapper.sh /app/main/myapp "$@"
```

For an extra-data app under `/app/extra/`, a `type: script` launcher can use:

```sh
export TMPDIR="${XDG_RUNTIME_DIR}/app/${FLATPAK_ID}"
exec zypak-wrapper /app/extra/myapp/myapp "$@"
```

Keep `TMPDIR` set for Chromium compatibility and verify the executable path
against the actual extracted layout.

## Desktop filename patching

Bundled `.asar` files sometimes hardcode the desktop filename, breaking taskbar
grouping and notifications. The BaseApp's `patch-electron-desktop-filename`
patches `resources/app.asar` to report the app ID. For binaries installed during
the build, run it after installation:

```yaml
build-commands:
  - install -Dm755 element.sh /app/bin/element
  - cp -r . /app/Element
  - patch-electron-desktop-filename ${FLATPAK_DEST}/Element/resources/app.asar
```

Older BaseApp branches such as `25.08` also provide `patch-desktop-filename`.
The `26.08` source branch provides only `patch-electron-desktop-filename`.
Check the selected BaseApp's commands before using either name.

Patching is relevant when a pre-built app uses Electron desktop integration
and upstream did not build its `.asar` with the Flatpak app ID. Extra-data
binaries arrive at install time, so a build command cannot patch those files;
adapt the install-time extraction flow if patching is needed.

## Sandbox notes

- Use zypak's sandbox support by default, rather than `--no-sandbox` or
  `ELECTRON_DISABLE_SANDBOX=1`.
- Use `ZYPAK_DISABLE_SANDBOX=1` only for a verified incompatibility and document
  why it is needed.
- Add `--allow=devel` only for functionality that needs it, such as a debugger
  using `ptrace`.
