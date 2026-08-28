# Mattermost Desktop Arch Package

This repository maintains the Arch Linux package for Mattermost Desktop. Keep `PKGBUILD` and `.SRCINFO` synchronized.

## Version updates

- Inspect the tagged upstream `package.json` before changing the package. Match `_electron` to the upstream Electron major and the `nodejs-lts-*` makedependency to the upstream Node engine requirement.
- Reset `pkgrel` to 1 for a new upstream version and update the GitHub archive checksum.
- Regenerate `.SRCINFO` with `makepkg --printsrcinfo` and verify that it exactly matches the checked-in file.

## npm and build details

- Node 24 currently brings npm 12. Keep npm arguments in a Bash array.
- Do not expand `$srcdir` in a top-level variable: makepkg has not defined it when it first sources `PKGBUILD`. Pass `--cache "$srcdir/npm-cache"` inside `prepare()` and `build()`.
- Keep `--allow-git=all`; upstream's `mmjstool` development dependency is fetched from a pinned Git commit, while npm 12 blocks Git dependencies by default.
- Do not add the old `config.target` or `config.runtime` fields to `package.json`. The package removes `electron-rebuild`, and npm 12 rejects those unknown project config keys in nested `npm run` calls.
- npm 12 may report blocked dependency install scripts. Electron Builder's `install-app-deps` rebuilds the native modules used by the build, and the upstream `beforePack` hook installs the platform-specific Koffi binary. Re-audit this if the blocked-package list or upstream hooks change.
- Electron Builder emits `app.asar.unpacked` with restrictive modes. Preserve the recursive `a+rX` normalization after copying it into `$pkgdir`, and inspect the package archive to ensure ordinary users can traverse directories and read runtime modules.

## Validation

- Run `makepkg --verifysource` and `diff -u .SRCINFO <(makepkg --printsrcinfo)`.
- Run `git diff --check`.
- When a system install is explicitly requested, validate with `makepkg --cleanbuild -si --noconfirm` and confirm the installed version using `pacman -Q mattermost-desktop`.
- The upstream build currently emits non-fatal Webpack size warnings and a makepkg warning that `app.asar` references `$srcdir`. Treat new warning categories or build errors as regressions to investigate.
