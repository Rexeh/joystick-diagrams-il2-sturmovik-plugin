# Joystick Diagrams — IL-2 Sturmovik Plugin

Parser plugin for [Joystick Diagrams](https://github.com/Rexeh/joystick-diagrams) that reads
IL-2 Sturmovik input exports (`global.actions` + `devices.txt`) and turns them into device
profiles the app can draw.

This plugin is **not bundled** with the application. Install it from inside Joystick Diagrams via
**Plugins → Store**, or point the installer at a release ZIP below.

- **Plugin ID:** `455dea21-ec69-4056-bc01-b9e00c6daf68`
- **Type:** parser
- **Current version:** 1.0.0

## Repository layout

```
il2_sturmovik_plugin/   # the importable plugin package — this folder is what ships in the release ZIP
  __init__.py
  main.py               # ParserPlugin entry point + IL2Settings
  il2_parser.py         # IL2Parser core logic
  img/icon.png
tests/                  # standalone tests + IL-2 fixtures
scripts/sign_plugin.py
```

## How it loads in the host

At runtime the host app imports this package and resolves `joystick_diagrams.*` against itself, so
imports of `joystick_diagrams.input.*`, `...plugin_interface`, and `...plugin_settings` stay
**absolute**. Intra-plugin imports (e.g. `from .il2_parser import IL2Parser`) are **relative**.

This plugin has **no third-party runtime dependencies** — it uses only the standard library and the
host-provided `joystick_diagrams` SDK.

## Development

```bash
# Install the host package (provides the joystick_diagrams SDK) plus dev tooling
pip install "joystick-diagrams @ git+https://github.com/Rexeh/joystick-diagrams@master"
pip install -e ".[dev]"

pytest tests/
ruff check .
```

## Releasing

Tag a release as `vX.Y.Z` (matching `plugin_meta.version` in `main.py`). The release workflow
zips the `il2_sturmovik_plugin/` folder, optionally signs it (see below), computes its SHA-256, and
publishes `il2_sturmovik_plugin.zip` as a release asset.

Then update the catalog manifest entry (`download_url` + `sha256`) that the app reads.

### Signing (for first-party "verified" releases)

The store shows this plugin as **Verified** only if the release ZIP contains a valid `plugin.sig`.
Signing uses the project's Ed25519 developer **private** key, stored as the repository/organisation
Actions secret `PLUGIN_SIGNING_KEY` (PEM). The matching public key is baked into the host
(`joystick_diagrams/plugins/plugin_signing.py`). Without the secret the workflow still ships an
**unsigned** ZIP (installs with a trust prompt).
