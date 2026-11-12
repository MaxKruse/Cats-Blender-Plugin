# Cats Blender Plugin (MaxKruse fork)

A community fork of [Cats Blender Plugin](https://github.com/teamneoneko/Cats-Blender-Plugin), picked up after the original NeoNeko version was archived at 5.3.1.0. This fork is an independent project maintained by **MaxKruse** - issues and pull requests go to [this repository](https://github.com/MaxKruse/Cats-Blender-Plugin), not to NeoNeko.

Cats is a Blender add-on for importing, fixing, optimizing, and exporting avatar models, especially for VRChat-style workflows. It is distributed as a Blender extension (package) for Blender 5.x.

## What it does

- Import helpers for common avatar/model formats, including MMD, XNALara/XPS, Source, FBX, VRM, and glTF/GLB workflows.
- Export helpers for checked avatar export and Resonite export.
- Armature repair tools for common imported rigs.
- Bone tools for root bones, bone merging, twist bones, digitigrade legs, and manual armature cleanup.
- Mesh cleanup tools for separating by materials, loose parts, or shapekeys.
- Shape key tools for pruning, repairing, applying shape keys to basis, and optimizing static shape keys.
- Viseme generation tools for avatar mouth shapes.
- Eye tracking setup tools for legacy and SDK3-style workflows.
- Material tools for combining duplicate materials and converting textures to PNG.
- Built-in texture atlas generator, with optional normal, emission, roughness, metallic, AO and height maps and Cycles baking.
- Translation tools for bones, shape keys, objects, and materials.
- Built-in model scaling, finger spreading and scale matching.
- Built-in updater that checks the releases of this repository.
- UI translations: English, Japanese, Korean, and Simplified Chinese.

## Blender version support

| Blender | Status |
|---|---|
| **5.2 and 5.3** | Supported |
| **5.1** | Use Cats 5.3.0.1 or earlier (final NeoNeko releases) |
| **5.4 and newer** | Not supported yet; Cats refuses to load |

## Install

Use the release zip. Do not unzip it before installing.

1. Download the latest `.zip` from the [Releases page](https://github.com/MaxKruse/Cats-Blender-Plugin/releases) (or build one yourself, see [Building](#building)).
2. Open Blender 5.2 or 5.3.
3. Go to `Edit > Preferences`.
4. Open `Get Extensions`.
5. Use the top-right drop-down menu and choose `Install from Disk`.
6. Select the downloaded zip.
7. Open `Add-ons`, search for `Cats`, and enable the add-on if Blender did not enable it automatically.
8. In the 3D Viewport, press `N` to open the sidebar and use the `CATS` tab.

Blender also exposes extension installation through `Preferences > Add-ons > Install from Disk`; either path is fine as long as you select the release zip.

Requirements:

- Blender from the [official Blender website](https://www.blender.org/download/) or Steam.
- Windows Store Blender is not supported (known permission issues).
- On Linux, package-manager, Snap and Flatpak builds of Blender are not supported.

Material Combiner and Immersive Scaler are not needed. Cats has its own built-in atlas generator and scaler.

## Updating

The updater panel in the `CATS` sidebar (`Settings & Updates`) checks the [releases of this repository](https://github.com/MaxKruse/Cats-Blender-Plugin/releases) and installs the newest release that supports your Blender version. Blender only installs a release whose manifest declares compatibility.

## Building

The repository is the extension source. To build a loadable zip:

1. Check out this repository.
2. Zip the repository contents (without `.git`) into a single top-level folder, e.g.:
   `maxkruse_cats/blender_manifest.toml`, `maxkruse_cats/__init__.py`, ...
3. Do not extract the zip before installing it in Blender.

`scripts/vendor_mmd_tools.py <version>` vendors a release of
[MMD Tools](https://github.com/MMD-Blender/blender_mmd_tools) into
`mmd_tools_local/` and lifts its `opencc` wheel into `wheels/`. After running it,
verify that the `wheels` list in `blender_manifest.toml` still matches the wheel on disk.

## Forking / contributing

- The development branch is `cats-latest`.
- Per-Blender-version branches (`5.2`, `5.3`, ...) exist so the in-app "download latest translations" feature can fetch `resources/translations/` and `resources/dictionary.json` for the running Blender version. When supporting a new Blender version, branch it off `cats-latest`.
- `CATS_VERSION` in `__init__.py` must stay in sync with `version` in `blender_manifest.toml`.
- See [AGENTS.md](AGENTS.md) for architecture notes aimed at contributors and AI agents.

## License

MIT. See [LICENSE](LICENSE).

## Credits

Cats Blender Plugin was originally developed by **Absolute Quantum**
([original project](https://github.com/absolute-quantum/cats-blender-plugin)) and then maintained and expanded by community contributors, most recently by **NeoNeko (Yusarina)** as the unofficial NeoNeko continuation, which is archived at 5.3.1.0. Thanks to all of them: this fork builds directly on that work.

Third-party projects Cats includes or builds on:

- [MMD Tools](https://github.com/MMD-Blender/blender_mmd_tools), bundled with Cats
- [Immersive Scaler](https://github.com/teamneoneko/immersive_scaler), whose scaling tools were ported into Cats
- [Material Combiner](https://github.com/Grim-es/material-combiner-addon), which earlier versions of Cats used for atlassing
