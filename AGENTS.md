# AGENTS.md - Cats Blender Plugin (MaxKruse fork)

Agent-facing notes for working in this repository. The user-facing doc is [README.md](README.md).

## Provenance / upstream

- This is a **community fork of [teamneoneko/Cats-Blender-Plugin](https://github.com/teamneoneko/Cats-Blender-Plugin)**, taken over from its `cats-latest` branch (commit `0b9947f`, version 5.3.1.0) after the NeoNeko project was archived.
- The original project is [absolute-quantum/cats-blender-plugin](https://github.com/absolute-quantum/cats-blender-plugin); the NeoNeko continuation (archived, but releases/wiki stay online) is at [git.neoneko.org/NeoNeko/Cats-Blender-Plugin](https://git.neoneko.org/NeoNeko/Cats-Blender-Plugin).
- `upstream` git remote = teamneoneko repo. Pull upstream changes from `cats-latest` only, deliberately - do not mirror the archived project.
- When in doubt about original behaviour, the NeoNeko wiki is the reference documentation.

## What this is

A **Blender 5.x extension (package)** for importing, fixing, optimizing and exporting avatar models (VRChat-style workflows). The repository root *is* the extension: `blender_manifest.toml` + `__init__.py` at the top level. A loadable artifact is just the repo contents (tracked files) zipped into one top-level folder; Blender installs it via "Install from Disk" without extracting.

## Identity and version invariants

- `blender_manifest.toml`:
  - `id = "maxkruse_cats"` - deliberately different from the archived extension's `cats_blender_plugin` id so Blender treats this as a separate extension (both can be installed side by side). Do not "fix" it back.
  - `version` (e.g. `5.3.2`) and `blender_version_min` / `blender_version_max` (5.2.0 / 5.4.0).
  - `wheels` lists bundled Python wheels; must stay in sync with the contents of `wheels/`.
- `CATS_VERSION` in `__init__.py` (e.g. `5.3.2.0`) **must equal** the manifest `version` plus a `.0` patch part. Release tags use the same `X.Y.Z.W` form.
- Upper Blender bound is enforced twice: `blender_version_max` in the manifest (Blender refuses to load) and `check_unsupported_blender_versions()` in `__init__.py` (runtime guard, currently blocks `>= (5, 4)`). Supporting a newer Blender means raising **both**, then testing.

## Layout

| Path | Role |
|---|---|
| `__init__.py` | Entry point. Loads all submodules (order matters, see below), register/unregister, version guard. |
| `globs.py` | Module-level globals (icons, `version_str`, `testing` operation log, `requests_available`). |
| `extentions.py` | Extension-specific glue. **The misspelling is intentional and referenced everywhere; do not rename.** |
| `updater.py` | Built-in updater. `RELEASES_API_URL` points at this repo's GitHub releases API; downloads the release zipball, extracts it, finds the dir containing `__init__.py`, overwrites the install in place (skips dotfiles, `resources/`, `downloads/`). |
| `tools/` | Feature logic (importer, armature, shapekey, viseme, translate, scaler, translations, settings, ...). |
| `ui/` | Blender panels per feature area. |
| `resources/` | Icons, `dictionary.json` (bone/shapekey translation dictionary), `translations/*.json` (UI strings, one file per language), valve resources. |
| `mmd_tools_local/` | **Vendored** MMD Tools (namespaced so it cannot clash with an official install). Shipped as-is. |
| `wheels/` | Bundled third-party wheels for Blender's Python (currently `opencc_python_reimplemented`). |
| `scripts/vendor_mmd_tools.py` | Vendors a MMD Tools release into `mmd_tools_local/` and lifts the opencc wheel into `wheels/`. Run as `python scripts/vendor_mmd_tools.py <version>`. |

## Module loading order

`__init__.py` imports submodules in a fixed order when Blender starts (`if "bpy" not in locals():`), and via `importlib.reload(...)` in the same order on add-on reload. Current order: `mmd_tools_local`, then `tools.*`, then `ui.*`, and `extentions` last. Keep new modules inside that sequence - several modules read each other's globals at import time.

Every subpackage `__init__.py` imports its submodules purely for the `register_wrap` side effect (defining a decorated class registers it). Those imports look unused to linters and are intentionally kept (`F401` is ignored for `__init__.py` in `ruff.toml`).

## Translations

- UI strings are addressed as `t('Section.key')` via `tools/translations.py`. `resources/translations/en_US.json` is canonical; `ja_JP`, `ko_KR`, `zh_CN` must carry the same keys. When changing a string's *meaning*, update all four files; when only rewording English, updating `en_US.json` alone is acceptable.
- "Download latest translations" (in-app operator) fetches `resources/translations/` and `resources/dictionary.json` through the GitHub contents/raw APIs from a **branch named after the running Blender minor version** (`translation_branch = "5.2"`, `"5.3", ...` in `tools/translations.py`). Supporting a new Blender version means that branch exists - it is created at release time (see Branches and Releasing).

## Branches and development workflow

- `cats-latest` - mainline and default branch. It must stay releasable at all times: supports exactly the Blender range in the manifest (currently 5.2-5.3), builds into a working zip, lints clean. Fix and feature work for the current supported line lands here (directly or via short-lived `fix/...` / `feat/...` branches cut from it).
- `5.2`, `5.3` - translation branches, content = `cats-latest` **at release time** (the in-app translation download reads them). Never develop on them; the only writes are fast-forwards to release commits (see Releasing, step 5).
- Experimental work for a **new Blender version** (e.g. 5.4) goes on a long-lived `feat/blender-<X.Y>` branch cut from `cats-latest` - it does not go on mainline while untested. While such a branch is open:
  - Do NOT raise `blender_version_max` in the manifest or the guard in `check_unsupported_blender_versions()`, and do NOT create the new translation branch - all three happen at release time so `cats-latest` keeps loading on (and only on) the supported range.
  - Keep the branch rebased on `cats-latest` so it picks up fixes.
  - When it is ready: merge into `cats-latest`, then release the new series (`X.Y.0.0`) - the release steps are where the version bump, upper-bound bump, guard bump, and `X.Y` translation branch all land together.
- `upstream` remote (teamneoneko) is a read-only reference - see Provenance.

## Gotchas

- `requests` may be missing (distro-packaged Blender). `__init__.py` tolerates that and sets `globs.requests_available = False`; network features must report `t('...noRequests')`-style errors instead of crashing. Core rig tools must keep working offline.
- `remove_corrupted_files()` in `__init__.py` deletes known-bad user files at load time; `extern_tools` in its list and in `.gitmodules` are leftovers - the immersive scaler code was ported into `tools/scaler/` and no submodule exists in this tree.
- The GitHub workflow `.github/workflows/Cats Tests.yml` is stale for this branch: it references `tests/run.py`, which does not exist here (the test suite lives on the NeoNeko Forgejo repo). Do not run it expecting green; if CI is revived, the tests need porting first.
- `.gitignore` excludes runtime/user data that may appear in the tree during use (`resources/settings.json`, `resources/downloads/`, `resources/icons/supporters/**`, crash logs, ...). Never commit those; never treat their absence as a bug.
- Windows Store Blender and Linux snap/flatpak/packaged Blender builds are unsupported (permission problems) - do not debug those environments.
- Vendored trees (`mmd_tools_local/`, `wheels/`) are excluded from linting; patching them locally is wrong, use `scripts/vendor_mmd_tools.py`.
- The updater's in-place update moves files into the extension's own install dir; anything not in the zip but present in the install dir (except dotfiles/`resources/`/`downloads/`) is deleted first.
- Eye test session state lives in module-scope globals in `tools/eyetracking.py` (`eye_left/right`, `*_data`, `*_rot`); a session is active iff all six are set (`_session_active()`). `scene.eye_mode` is a UI mirror that persists into .blend files but the session does not survive a reload - `sync_eye_mode_on_load` (load_post, registered in `__init__.py`) resets a stale 'TESTING' to 'CREATION' and drops dead session refs. All teardown goes through `_end_session()`; do not add new code that mutates the session globals directly.
- `FixArmature.execute` tags bones with `cats_orig_name` via `Common.snapshot_bone_identities`; the tag must be cleared with `Common.clear_bone_identity_snapshot` before every return after the snapshot (the early CANCELLED paths included), so it never reaches a saved file or export.

## Lint / format

`ruff.toml` is configured (Pyflakes rules `F` only, `target-version = "py313"` - Blender 5.2 bundles CPython 3.13). Lint with:

```
uvx ruff check .
```

## Building a release zip

1. Make sure the working tree is clean and `CATS_VERSION` == manifest `version` + `.0`.
2. Stage the tracked files (e.g. `git archive`) into a single top-level folder named after the manifest id (`maxkruse_cats/`), including `wheels/` and `blender_manifest.toml`.
3. Zip that folder. Users install the zip directly (no extracting) via `Preferences > Get Extensions > Install from Disk`.

## Releasing

Releases are GitHub releases on **this** repo: annotated tag + release + zip asset. The in-app updater reads `tag_name`, `body`, `published_at` from the releases API and installs from the release's `zipball_url` (the repo tree at the tagged commit) - the uploaded asset exists only for manual downloads. So the tag must point at the exact commit that was built and tested.

1. Bump the version in **both** places and commit: `version` in `blender_manifest.toml` (e.g. `5.3.3`) and `CATS_VERSION` in `__init__.py` (e.g. `5.3.3.0` - always the manifest version plus `.0`).
2. Build the zip (above) → `dist/maxkruse_cats-<X.Y.Z.W>.zip`.
3. Annotated tag with the full four-part version (the updater compares dotted ints, so three-part tags sort wrong):
   `git tag -a <X.Y.Z.W> -m "<X.Y.Z.W> - <summary>" && git push origin <X.Y.Z.W>`
   Include the `Signed-by: omp (<provider> <model>)` trailer in the tag message.
4. Create the release:
   `gh release create <X.Y.Z.W> --repo MaxKruse/Cats-Blender-Plugin --title "<X.Y.Z.W>" --target cats-latest --notes "<what changed>" dist/maxkruse_cats-<X.Y.Z.W>.zip`
   - `--repo` is mandatory: in this clone `gh` resolves the default repo from the `upstream` remote (the archived teamneoneko repo) and the create fails with a 404.
5. Fast-forward the translation branches to the release commit so in-app "download latest translations" serves the new strings:
   `git push origin <X.Y.Z.W>:refs/heads/5.2 <X.Y.Z.W>:refs/heads/5.3`
6. Verify: `gh api repos/MaxKruse/Cats-Blender-Plugin/releases` shows the release with `tag_name`, `zipball_url`, `published_at` and the zip asset.

Version scheme: `.W` patches are fixes (`5.3.2.1`), `.Z` minors are features (`5.3.3.0`). Adding support for a new Blender version is a new series (`5.4.0.0`) and additionally requires raising `blender_version_max` in the manifest, the guard in `check_unsupported_blender_versions()` in `__init__.py`, testing on that Blender, and creating the matching translation branch (e.g. `5.4`) at release time.
