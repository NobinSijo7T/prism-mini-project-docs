# Prism-Browser-Template

Small, GitHub-friendly template for maintaining Prism changes on top of Chromium without pushing the full Chromium source tree.

##Check [Contributor section for easy and fast development](https://github.com/PrasanthPradeep/prism-chromium/tree/main?tab=readme-ov-file#for-contributors)

## Why this approach

- Keeps your public repository small
- Tracks only Prism-specific modifications
- Rebuilds from a known Chromium base revision
- Avoids trying to host Chromium's full history on GitHub

## Layout

```text
prism-browser-template/
├── BASE_REVISION.txt
├── patches/
│   ├── 0001-*.patch
│   └── *.diff
└── scripts/
  ├── apply_patches.sh
  └── build_prism.sh
```

## 1) Capture your Prism changes

From your Chromium checkout (`~/dev/prism/src`):

### Preferred: commit and export with `format-patch`

```bash
# commit your Prism work first (one or many commits)
git status
git add -A
git commit -m "Prism: <feature>"

# export commit(s) into template repo
mkdir -p /path/to/prism-browser-template/patches
git format-patch -N -o /path/to/prism-browser-template/patches <upstream-base>..HEAD
```

### Fallback: working tree diff (includes binaries)

```bash
git status --untracked-files=all
git diff --binary --full-index > /path/to/prism-browser-template/patches/0001-prism-working-tree.diff
```

## 2) Record Chromium base revision

```bash
cd ~/dev/prism/src
git rev-parse HEAD > /path/to/prism-browser-template/BASE_REVISION.txt
```

This SHA is used by the apply script to prevent applying patches onto the wrong Chromium revision.

## 3) Apply patches onto a fresh Chromium checkout

```bash
# example bootstrap
fetch chromium
cd chromium
gclient sync
cd src

# apply prism patch series
/path/to/prism-browser-template/scripts/apply_patches.sh
```

Useful options:

```bash
/path/to/prism-browser-template/scripts/apply_patches.sh --check
/path/to/prism-browser-template/scripts/apply_patches.sh --skip-base-check
/path/to/prism-browser-template/scripts/apply_patches.sh --patch-dir /custom/patch/dir
```

## 4) One-command patch + build

Run from `chromium/src`:

```bash
/path/to/prism-browser-template/scripts/build_prism.sh
```

Useful options:

```bash
/path/to/prism-browser-template/scripts/build_prism.sh --check-only
/path/to/prism-browser-template/scripts/build_prism.sh --skip-sync
/path/to/prism-browser-template/scripts/build_prism.sh --target chrome --out-dir out/Prism
/path/to/prism-browser-template/scripts/build_prism.sh --gn-args 'is_debug=false symbol_level=1'
```

## 5) Build Chromium with Prism changes (manual)

```bash
gn gen out/Default
ninja -C out/Default chrome
```

## Suggested GitHub workflow

- Keep this template repository public/small
- Keep Chromium source local (or use upstream remote)
- Add a CI job that:
  1. syncs Chromium
  2. applies patches
  3. builds/tests Prism

## For Contributors

Use these steps on a fresh machine to reproduce Prism from this repository.

### Prerequisites

- Linux/macOS (or Windows with equivalent depot_tools setup)
- `depot_tools` installed and available in `PATH`

### Quickstart

```bash
# 1) Get Chromium source
fetch chromium
cd chromium
gclient sync

# 2) Enter src
cd src

# 3) Apply Prism patches (from this repo)
/path/to/prism-browser-template/scripts/apply_patches.sh

# 4) Build
gn gen out/Default
ninja -C out/Default chrome
```

### Optional one-command flow

```bash
# From chromium/src
/path/to/prism-browser-template/scripts/build_prism.sh --skip-sync
```

### Notes

- `BASE_REVISION.txt` pins the expected Chromium commit.
- If your checkout is at a different commit, re-sync to the base or run with
  `--skip-base-check` only if you intentionally accept patch drift.

## Notes

- `*.patch` files are applied before `*.diff` files.
- Keep patch files focused by feature (`0001-ui.patch`, `0002-ai-button.patch`, ...).
- If base revision drifts, regenerate patches against the new Chromium base.
