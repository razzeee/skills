# MetaInfo and desktop files

Flathub requires metadata in the upstream project unless the app uses
extra-data. For that exception, keep missing files alongside the manifest and
install them via repository-relative `type: file` sources. Install MetaInfo to
`/app/share/metainfo/<AppID>.metainfo.xml` and validate it with the builder's
`flatpak-builder-lint appstream` command.

## MetaInfo template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component type="desktop-application">
  <id>io.github.alice.MyApp</id>

  <metadata_license>CC0-1.0</metadata_license>
  <project_license>GPL-3.0-only</project_license>

  <name>My App</name>
  <summary>A short one-line description (no trailing period)</summary>

  <developer id="io.github.alice">
    <name>Alice</name>
  </developer>

  <description>
    <p>A paragraph describing what the app does.</p>
    <ul>
      <li>Feature one</li>
      <li>Feature two</li>
    </ul>
  </description>

  <launchable type="desktop-id">io.github.alice.MyApp.desktop</launchable>

  <branding>
    <color type="primary" scheme_preference="light">#4a90d9</color>
    <color type="primary" scheme_preference="dark">#2c5f8a</color>
  </branding>

  <!-- Pick brand colors that match the app's visual identity.
       Preview them at https://docs.flathub.org/banner-preview -->

  <content_rating type="oars-1.1" />

  <url type="homepage">https://github.com/alice/myapp</url>
  <url type="bugtracker">https://github.com/alice/myapp/issues</url>
  <url type="vcs-browser">https://github.com/alice/myapp</url>

  <screenshots>
    <screenshot type="default">
      <image>https://raw.githubusercontent.com/alice/myapp/v1.2.3/screenshots/main.png</image>
      <caption>Main window</caption>
    </screenshot>
  </screenshots>

  <releases>
    <release version="1.2.3" date="2025-01-15">
      <description>
        <p>Initial Flathub release.</p>
      </description>
    </release>
  </releases>

  <requires>
    <control>keyboard</control>
    <control>pointing</control>
    <display_length compare="ge">768</display_length>
  </requires>
</component>
```

## MetaInfo checklist

- Match `<id>` to the manifest's app ID exactly.
- `<metadata_license>` licenses the XML itself; use `CC0-1.0` for newly
  authored metadata where appropriate, preserving existing license notices.
- Use the app's SPDX identifier in `<project_license>`; proprietary apps use
  a license reference such as `LicenseRef-proprietary=https://…`.
- Give `<developer>` a reverse-DNS `id` attribute.
- Use direct screenshot image URLs pinned to a tag or commit.
- Include actual release versions and dates, never future dates.
- Generate OARS ratings at <https://hughsie.github.io/oars/generate.html>.
- Flathub's website supplies the community-package disclaimer; no extra
  description paragraph is required for it.
- AppStream descriptions use XML markup. Flathub also supports `**...**` around
  an entire paragraph for [website-specific emphasis](https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines#website-specific-formatting).

## Desktop file

Install `<AppID>.desktop` to `/app/share/applications/`:

```ini
[Desktop Entry]
Name=My App
Comment=A short one-line description
Exec=myapp
Icon=io.github.alice.MyApp
Terminal=false
Type=Application
Categories=Utility;
```

- Set `Icon` to the app ID without a file extension.
- Use valid freedesktop Menu Spec categories. Include an appropriate main
  category such as `Utility`; toolkit categories alone do not describe the app.
