# Kvrocks Distro Packages by FPM

Generate `.deb` / `.rpm` packages for [Apache Kvrocks](https://kvrocks.apache.org) via [FPM](https://github.com/jordansissel/fpm) and GitHub Actions.

This is a fork of [RocksLabs/kvrocks-fpm](https://github.com/RocksLabs/kvrocks-fpm) with:

- Bumped to the latest Apache Kvrocks release (see `VERSION`).
- Refactored CI into a reusable workflow (`_build.yaml`) shared by manual and automated builds.
- Daily auto-update workflow (`auto-update.yaml`) that watches `apache/kvrocks` releases, bumps `VERSION`, commits, and triggers a fresh build.

## How it works

| File | Purpose |
| --- | --- |
| `VERSION` | Upstream Apache Kvrocks version to build (e.g. `2.15.0`). |
| `ITERATION` | Package revision (bumped manually for repackages of the same upstream version). |
| `.github/workflows/_build.yaml` | Reusable workflow: builds Kvrocks from source, packages `.deb` and `.rpm` via FPM, attaches them to a GitHub Release. |
| `.github/workflows/ci.yaml` | Triggered by pushing a tag matching `2.*` or via manual dispatch. Reads `VERSION` / `ITERATION` and calls the reusable build. |
| `.github/workflows/auto-update.yaml` | Cron (daily 04:17 UTC). Checks the latest `apache/kvrocks` release, bumps `VERSION`, commits, and immediately calls the reusable build. Also supports manual dispatch with a `force_version` input. |

## Manual release

```sh
echo "2.15.0" > VERSION
echo "1"      > ITERATION
git commit -am "Bump to 2.15.0"
git tag 2.15.0-1
git push origin master --tags
```

The `2.*` tag triggers `ci.yaml`, which builds and publishes the release.

## Force-build a specific version

GitHub Actions -> **Auto-update Apache Kvrocks** -> **Run workflow** -> set `force_version` to e.g. `2.14.0`.

This **force-builds** the specified upstream version using the current `ITERATION`. It does **not** modify the `VERSION` file in master. The release is published with a unique tag `<version>-<iteration>-manual-<runId>` to avoid clashing with regular releases.

## Credits

Original packaging by [RocksLabs/kvrocks-fpm](https://github.com/RocksLabs/kvrocks-fpm). Apache Kvrocks is a project of the Apache Software Foundation.
