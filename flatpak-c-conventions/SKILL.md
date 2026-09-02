---
name: flatpak-c-conventions
description: Apply Flatpak's C integer type conventions. Use whenever writing, editing, fixing, refactoring, or reviewing C code in the Flatpak repository, especially loops, array indexes, collection lengths, fixed-width integers, and GLib integer typedefs.
---

# Flatpak C conventions

- Prefer standard C integer types over GLib typedefs when either is suitable. For example, use `size_t`, `uint8_t`, `uint32_t`, and `int64_t` instead of `gsize`, `guint8`, `guint32`, and `gint64`.
- Use `size_t` for array indexes, collection lengths, and loop counters compared with a size or length.
- Keep GLib types when required by a GLib API, when changing the type would alter its signedness or width, or when matching an existing public API.
