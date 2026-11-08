# Plugin and extension addons

Some apps expose an extension point for separately packaged plugins, such as
OBS Studio plugins, LibreOffice macros, or GIMP plug-ins.

## Parent app: declare the extension point

Add `add-extensions` to the parent manifest and create its target directory:

```yaml
add-extensions:
  com.example.MyApp.Plugin:
    version: stable
    directory: plugins
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

`directory` is relative to `/app`. Configure `merge-dirs` for the paths that
must be merged into the host's view, relative to each plugin's prefix.
Incorrect paths prevent the host from finding installed plugin files.

## Plugin manifest

Use the SDK branch the parent actually uses; `6.9` below is only an example.

```yaml
id: com.example.MyApp.Plugin.FooPlugin
branch: stable
runtime: com.example.MyApp
runtime-version: stable
sdk: org.kde.Sdk//6.9
build-extension: true
separate-locales: false

build-options:
  prefix: /app/plugins/FooPlugin

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

Rules:

- Set `runtime` to the parent app ID, not a Platform runtime.
- Match the parent's SDK to avoid ABI mismatches and linker errors.
- Set `build-extension: true`.
- Put `prefix` inside the parent's extension directory.
- Inherit the parent's sandbox; omit `finish-args` from the extension manifest.
- Bundle dependencies missing from the parent's SDK before the plugin module.

## Extension MetaInfo

Use `type="addon"` and declare the parent in `<extends>`:

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

Addon metadata does not need `<launchable>`, `<screenshots>`, or `<branding>`
from the desktop application's template.
