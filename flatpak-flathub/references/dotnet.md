# .NET / NuGet

Select an available `org.freedesktop.Sdk.Extension.dotnet*` extension that
supports the application's target framework. Match the extension branch to
the underlying Freedesktop SDK. Versions below are examples, not current
release recommendations.

## Generate NuGet sources

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
cd flatpak-builder-tools/dotnet
# Install the matching Freedesktop SDK and .NET SDK extension first.
python3 flatpak-dotnet-generator.py nuget-sources.json MyApp.csproj \
  --runtime linux-x64 --freedesktop 25.08 --dotnet 9
```

Check the generator's current installation instructions and flags. Pass
`--dotnet` explicitly to match the SDK extension; the generator's default may
be a different major version. Match the runtime identifier to each target
architecture and check that restore succeeds.

## Manifest

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.dotnet9

finish-args:
  - --env=DOTNET_ROOT=/app/lib/dotnet

modules:
  - name: dotnet
    buildsystem: simple
    build-commands:
      - /usr/lib/sdk/dotnet9/bin/install.sh

  - name: myapp
    buildsystem: simple
    build-options:
      append-path: /usr/lib/sdk/dotnet9/bin
      env:
        DOTNET_CLI_TELEMETRY_OPTOUT: 'true'
        DOTNET_NOLOGO: 'true'
    build-commands:
      - |
        . /usr/lib/sdk/dotnet9/enable.sh
        dotnet publish -c Release --no-self-contained --source nuget-sources -o /app/lib/myapp MyApp.csproj
      - |
        install -Dm755 /dev/stdin ${FLATPAK_DEST}/bin/myapp <<'EOF'
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

Source the SDK environment and run the build in the same shell command so
environment changes survive. Use YAML literal blocks for multiline shell
commands and heredocs.

The SDK extension is available only during builds. For framework-dependent
publishing, `install.sh` copies its runtime into `/app/lib/dotnet`, and
`DOTNET_ROOT` tells the apphost where to find it. This follows the
[Flatpak .NET guide](https://docs.flatpak.org/en/latest/dotnet.html#creating-the-flatpak).
Merge the example's environment setting with the app's other finish arguments.

Pass `--source nuget-sources` to `dotnet build` / `dotnet publish` for offline
sources. Regenerate whenever `.csproj` or `packages.lock.json` changes.
