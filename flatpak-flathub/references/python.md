# Python dependencies

Python 3 is included in GNOME, KDE, and Freedesktop runtimes, so pure Python
apps do not need a separate Python SDK extension. Verify the Python version
in the selected runtime against the application's requirements.

## Generate offline sources

Build steps cannot fetch pip dependencies. Use `flatpak-pip-generator` from
[flatpak-builder-tools](https://github.com/flatpak/flatpak-builder-tools/tree/master/pip)
to resolve wheels and sdists into a source manifest.

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
cd flatpak-builder-tools/pip
uv run flatpak-pip-generator.py --help
```

Use `uv run` to install the script's declared dependencies, including
`requirements-parser` and `packaging`. The script requires Python 3.10 or newer.
Check its installation instructions and `--help` before running it.
The commands below illustrate supported input forms; use
the actual SDK branch selected for the application instead of the example `49`.

From `requirements.txt`:

```bash
uv run flatpak-pip-generator.py \
  --runtime org.gnome.Sdk//49 \
  --requirements-file requirements.txt \
  --output python3-modules
```

From `pyproject.toml`:

```bash
uv run flatpak-pip-generator.py \
  --runtime org.gnome.Sdk//49 \
  --pyproject-file pyproject.toml \
  --output python3-modules
```

For compiled packages where platform wheels are needed:

```bash
uv run flatpak-pip-generator.py \
  --runtime org.gnome.Sdk//49 \
  --prefer-wheels=grpcio,cryptography \
  --requirements-file requirements.txt \
  --output python3-modules
```

Match `--runtime` to the SDK used by the manifest so resolution uses the right
Python version. Verify generated sources cover each target architecture.

## Include the generated modules

```json
{
  "modules": [
    "python3-modules.json",
    {
      "name": "myapp",
      "buildsystem": "meson"
    }
  ]
}
```

Or YAML:

```yaml
modules:
  - python3-modules.json
  - name: myapp
    buildsystem: meson
```

Regenerate when dependencies change and commit `python3-modules.json` alongside
the manifest. Include the generation command in the README's Update section:

````markdown
### Updating Python dependencies

```bash
git clone https://github.com/flatpak/flatpak-builder-tools.git
uv run flatpak-builder-tools/pip/flatpak-pip-generator.py \
  --runtime org.gnome.Sdk//49 \
  --requirements-file requirements.txt \
  --output python3-modules
```
````
