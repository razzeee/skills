# Go / Go Modules

Add `org.freedesktop.Sdk.Extension.golang` to `sdk-extensions`, using a branch
compatible with the underlying Freedesktop SDK.

## Generate vendored sources

```bash
go install github.com/dennwc/flatpak-go-mod@latest
flatpak-go-mod ./path/to/go/project
```

This produces `go.mod.yaml` and `modules.txt`. Check the tool's current usage
and run it against the directory containing `go.mod`.

## Manifest

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.golang

modules:
  - name: myapp
    buildsystem: simple
    build-options:
      append-path: /usr/lib/sdk/golang/bin
      env:
        GOBIN: /app/bin
    build-commands:
      - go install -mod=vendor ./cmd/myapp
    sources:
      - type: git
        url: https://github.com/example/myapp
        commit: FULL_COMMIT_SHA
      - type: file
        path: modules.txt
        dest: vendor
      - go.mod.yaml
```

Set `GOBIN` so the executable is installed into the application prefix.
Regenerate whenever `go.mod` or `go.sum` changes and commit both generated files.
