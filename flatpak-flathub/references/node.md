# Node / npm / yarn / pnpm / Electron

## Generate offline sources

```bash
pipx install 'git+https://github.com/flatpak/flatpak-builder-tools.git#subdirectory=node'

# npm
flatpak-node-generator npm package-lock.json -o generated-sources.json

# yarn
flatpak-node-generator yarn yarn.lock -o generated-sources.json

# pnpm
flatpak-node-generator pnpm pnpm-lock.yaml -o generated-sources.json
```

Check the installed generator's `--help` for supported lockfile formats and
flags. If the output exceeds GitHub's file-size limit, use the generator's
split option (`-s`) to produce numbered source files.

## SDK extension

Select an available Node extension compatible with the app and the underlying
Freedesktop SDK. For example, `org.freedesktop.Sdk.Extension.node22//25.08`
uses `append-path: /usr/lib/sdk/node22/bin`. Verify support before copying the
example; Node versions and branches change independently.

## Electron build details

- Set `ELECTRON_CACHE` and `npm_config_cache` in `build-options.env` to the
  generated `flatpak-node/` locations so prebuilt binaries are available offline.
- For ARM/ARM64 electron-builder, source
  `flatpak-node/electron-builder-arch-args.sh` to set `$ELECTRON_BUILDER_ARCH_ARGS`.
- For native addon ABI compatibility, use the generator's
  `--electron-node-headers` option and set `npm_config_nodedir` to
  `flatpak-node/node-gyp/electron-current` as supported by that generator version.
- Use specific D-Bus names rather than session-bus access.
- Apply the Electron runtime and launcher guidance linked from `SKILL.md`.

Regenerate whenever the lockfile changes. Commit `generated-sources.json`
or all numbered source files when splitting.
