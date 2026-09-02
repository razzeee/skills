---
name: glib-c-conventions
description: Apply C type conventions used in Mutter, xdg-desktop-portal, Flatpak, and similar GLib-based projects. Use whenever writing, editing, fixing, refactoring, or reviewing their C code, especially loops, array indexes, collection lengths, fixed-width integers, and GLib typedefs.
---

# GLib C type conventions

- Prefer standard C types over GLib aliases that add no meaning. For example, use `char`, `size_t`, `uint8_t`, `uint32_t`, and `int64_t` instead of `gchar`, `gsize`, `guint8`, `guint32`, and `gint64` in new code.
- Use `size_t` for array indexes, collection lengths, and loop counters compared with a size or length.
- Preserve `gsize` when it is already established in older surrounding code; do not introduce it for new index or length variables.
- Use a GLib type when it carries useful semantics or is needed to match an existing public API, callback signature, or surrounding interface. Do not replace types mechanically when that would alter signedness, width, or compatibility.
