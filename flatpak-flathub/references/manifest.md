# Manifest rules and template

Name the manifest `<AppID>.json` or `<AppID>.yaml` at the repository root.
Replace the example runtime branch with a verified supported branch.

## Minimal JSON template

```json
{
  "app-id": "io.github.alice.MyApp",
  "runtime": "org.gnome.Platform",
  "runtime-version": "49",
  "sdk": "org.gnome.Sdk",
  "command": "myapp",
  "finish-args": [
    "--share=ipc",
    "--socket=fallback-x11",
    "--socket=wayland",
    "--device=dri"
  ],
  "cleanup": [
    "/include",
    "/lib/pkgconfig",
    "*.la", "*.a"
  ],
  "modules": [
    {
      "name": "myapp",
      "buildsystem": "meson",
      "sources": [
        {
          "type": "git",
          "url": "https://github.com/alice/myapp.git",
          "tag": "v1.2.3",
          "commit": "<full sha>"
        }
      ]
    }
  ]
}
```

## Key manifest rules

- Keep all files in the packaging repository. Local source paths are relative
  to the manifest that declares them; keep them inside the repository.
- Build steps have no network access. Adding `--share=network` to `build-args`
  does not enable downloads; declare sources with public URLs.
- Package-manager dependencies need pre-generated offline sources. Use the
  applicable language reference linked from `SKILL.md` and
  [flatpak-builder-tools](https://github.com/flatpak/flatpak-builder-tools).
- Use the `shared-modules` git submodule for common dependencies such as
  `libayatana-appindicator`, `lua`, and `SDL_sound`.
- Install license files to `$FLATPAK_DEST/share/licenses/$FLATPAK_ID/`.
- Put the main app module last, and frequently changing modules last within
  their group, to reduce rebuild work.
- Use `rename-icon`, `rename-desktop-file`, and `rename-appdata-file` when
  upstream filenames do not already carry the app ID prefix.

## finish-args quick reference

Request only what the app needs:

```text
# Display
--share=ipc                     # Pair with X11
--socket=fallback-x11           # X11 fallback, used with Wayland
--socket=wayland                # Native Wayland
--device=dri                    # GPU / OpenGL

# Audio
--socket=pulseaudio             # Sound I/O

# Network
--share=network                 # Internet access at runtime

# Filesystem (prefer portals over blanket access)
--filesystem=xdg-documents
--filesystem=xdg-download
--filesystem=xdg-music:ro
--filesystem=home               # Avoid when portals or narrower access suffice

# D-Bus (specific names instead of session-bus access)
--talk-name=org.freedesktop.Notifications
--own-name=org.mpris.MediaPlayer2.myapp
```

Prefer `xdg-desktop-portal` for file access. Grant all-device access only when
the application demonstrably needs it.
