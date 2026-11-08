# Rust / Cargo

Add `org.freedesktop.Sdk.Extension.rust-stable` to `sdk-extensions`. Match its
branch to the underlying Freedesktop SDK version. GNOME and KDE runtime
numbers are not the corresponding Freedesktop branch numbers.

## Generate cargo-sources.json

Check the generator's current installation instructions, then generate sources
from the application's lockfile. Example using Poetry:

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
cd flatpak-builder-tools/cargo
pip install poetry
poetry install
poetry run python3 flatpak-cargo-generator.py /path/to/Cargo.lock -o cargo-sources.json
```

## Simple buildsystem

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

## Meson-managed CARGO_HOME

If `meson.build` sets `CARGO_HOME` itself, Cargo may not find the generated
configuration. Copy it to the project's `.cargo/config.toml` when needed:

```yaml
buildsystem: meson
build-options:
  append-path: /usr/lib/sdk/rust-stable/bin
  env:
    CARGO_NET_OFFLINE: 'true'
sources:
  - type: git
    url: https://github.com/example/myapp
    commit: FULL_COMMIT_SHA
  - cargo-sources.json
  - type: shell
    commands:
      - mkdir -p .cargo
      - cp -vf cargo/config .cargo/config.toml
```

Regenerate whenever `Cargo.lock` changes and commit `cargo-sources.json`.
