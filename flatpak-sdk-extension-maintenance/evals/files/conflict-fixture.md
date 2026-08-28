# Conflict Fixture

Create the fixture under the run's output directory. It is intentionally local
and must not be pushed anywhere.

1. Initialize a Git repository with user name `Eval User` and email
   `eval@example.invalid`.
2. Commit these files on `branch/25.08`:

`org.example.Extension.yaml`:

```yaml
app-id: org.example.Extension
branch: "25.08"
runtime: org.freedesktop.Sdk
runtime-version: "25.08"
build-options:
  strip: true
modules:
  - name: example
    sources:
      - type: archive
        url: file://SOURCE_V1_PATH
        sha256: SOURCE_V1_SHA256
```

`org.example.Extension.metainfo.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component type="addon">
  <id>org.example.Extension</id>
  <releases>
    <release version="1.0.0" date="2026-01-01"><description/></release>
  </releases>
</component>
```

3. Branch `branch/24.08` from that commit. On `branch/24.08`, change only the
   manifest identity to `branch: "24.08"`, `runtime-version: "24.08"`, and add
   `prepend-path: /usr/lib/sdk/example/bin` under `build-options`; commit it.
4. Return to `branch/25.08`. Create a local archive named
   `example-1.1.0.tar.xz`, calculate its SHA-256, update the manifest URL and
   checksum, and prepend a `1.1.0` release dated `2026-02-02`; commit as
   `example: Update 1.0.0 to 1.1.0`. Tag this commit `reference-update`.
5. Branch `branch/26.08` from `reference-update`. Change only the manifest
   identity to `branch: "26.08"` and `runtime-version: "26.08beta"`; commit it.
6. Treat `reference-update` as the supplied reference PR head and
   `branch/25.08` as its base. The archive's final absolute `file://` URL and
   checksum are the verified source data.

The intended outcome is:

- `branch/25.08` and `branch/26.08` are already current.
- `branch/24.08` receives release 1.1.0 by cherry-pick.
- Its `branch: "24.08"`, `runtime-version: "24.08"`, and `prepend-path` survive
  conflict resolution unchanged.
- The archive checksum validates and each release appears exactly once.
