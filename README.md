# dnd-mapp/action-verify-release

[![push main](https://github.com/dnd-mapp/action-verify-release/actions/workflows/push-main.yaml/badge.svg?branch=main)](https://github.com/dnd-mapp/action-verify-release/actions/workflows/push-main.yaml)
[![license](https://img.shields.io/github/license/dnd-mapp/action-verify-release)](LICENSE)

Composite GitHub Action that checks a pushed release tag against `package.json` and `CHANGELOG.md`, and writes the release notes.

The D&D Mapp packages are released from a tag push. This action runs first, in a job without side effects, and fails on anything that would make the release wrong. The jobs after it stage the package on npm and create the GitHub Release with plain steps. See [The release workflow](#the-release-workflow).

## Requirements

- A workflow that runs on `push` of release tags only, such as `v[0-9]+.[0-9]+.[0-9]+`. The action trusts the trigger and reads the version from the tag name without the leading `v`.
- `@dnd-mapp/changelog-tools` as a dev dependency of the repository, installed before the action runs. The action calls `pnpm exec changelog`.
- A checkout of the tagged commit. A shallow checkout is fine, because the action fetches the base branch itself.

## Usage

Pin the action to a commit SHA and note the version in a comment, like the third-party actions in the D&D Mapp workflows. Take the SHA from the commit that the release tag points to.

```yaml
- name: Verify the tag and the changelog
  id: release
  uses: dnd-mapp/action-verify-release@<commit-sha> # v1.0.0
```

Run it after the dependencies are installed, and upload the notes file as an artifact so the job that creates the GitHub Release can download it.

## What it checks

1. The tagged commit is reachable from `origin/<base-branch>`. This replaces the branch check that `--no-git-checks` turns off when the package is staged.
2. `changelog verify --version X.Y.Z` passes. It checks the version against `package.json` and the section, its date, its entries, and the link references in the changelog.
3. `changelog notes --version X.Y.Z --output <notes-file>` writes the release notes.

The action does not check the tag name itself. A tag that is not a release version fails in `changelog verify`, because the version is not valid SemVer or has no section.

## Inputs

| Input         | Default            | Description                                            |
|:--------------|:-------------------|:-------------------------------------------------------|
| `changelog`   | `CHANGELOG.md`     | Path to the changelog, relative to the repository root |
| `notes-file`  | `release-notes.md` | Path of the release notes file to write                |
| `base-branch` | `main`             | Branch that the tagged commit must be reachable from   |

## Outputs

| Output       | Description                                       |
|:-------------|:--------------------------------------------------|
| `version`    | The version from the tag, without the leading `v` |
| `notes-file` | Path of the release notes file that was written   |

## The release workflow

Every package repository has this `release.yaml`. It is also the workflow that the npm trust relationship names, so keep its filename. The `./.github/actions/ci` step stands for the checks of the repository, and it installs the dependencies.

```yaml
name: Release

on:
    push:
        tags:
            - "v[0-9]+.[0-9]+.[0-9]+"

permissions: {}

jobs:
    verify:
        name: Verify
        runs-on: ubuntu-26.04
        timeout-minutes: 10
        permissions:
            contents: read
        steps:
            - name: Checkout repository
              uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

            - name: Run CI checks
              uses: ./.github/actions/ci

            - name: Verify the tag and the changelog
              uses: dnd-mapp/action-verify-release@<commit-sha> # v1.0.0

            - name: Upload the release notes
              uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
              with:
                  name: release-notes
                  path: release-notes.md
                  if-no-files-found: error

    publish:
        name: Stage on npm
        needs: verify
        runs-on: ubuntu-26.04
        timeout-minutes: 10
        environment: npm
        permissions:
            contents: read
            id-token: write
        steps:
            - name: Checkout repository
              uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

            - name: Set up Node.js and pnpm and install dependencies
              uses: pnpm/setup@703c52620218391530e48b9e8870d5c0082e1b9b # v2.1.0
              with:
                  cache: true
                  runtime: node
                  require-lockfile: true

            - name: Stage the package
              shell: bash
              run: pnpm stage publish --access public --provenance --tag latest --no-git-checks

    release:
        name: GitHub Release
        needs: [verify, publish]
        runs-on: ubuntu-26.04
        timeout-minutes: 5
        permissions:
            contents: write
        steps:
            - name: Download the release notes
              uses: actions/download-artifact@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c # v8.0.1
              with:
                  name: release-notes

            - name: Create the release
              shell: bash
              run: gh release create "$GITHUB_REF_NAME" --verify-tag --title "Release $GITHUB_REF_NAME" --notes-file release-notes.md
              env:
                  GH_TOKEN: ${{ github.token }}
                  GH_REPO: ${{ github.repository }}
```

A maintainer approves the staged version afterwards with `pnpm stage approve <id>` and 2FA. Use `pnpm stage list` to find it, and `pnpm stage reject <id>` when something is wrong, because the same version cannot be staged again until then.

### Why it looks like this

- Three jobs, so the npm token and the write access to the repository each exist in one job only. Everything that can fail for a fixable reason runs in `verify`, before any side effect.
- `--no-git-checks`, because a tag checkout is a detached HEAD and pnpm would refuse to publish. The ancestry check of this action is the stricter replacement.
- `environment: npm`, so the OIDC token carries an environment claim that the trust relationship pins. Protection rules can be added later without touching the workflow.
- The GitHub Release comes after staging. A failed stage leaves no half-finished release behind, and GitHub Releases in the D&D Mapp repositories are immutable.
- Plain steps for staging and the release, because each is one command. A composite action and not a reusable workflow for the checks, because npm trusted publishing validates the filename of the calling workflow.

### One-time npm setup

Run this once per package from a shell that is logged in to npm with 2FA. The trust relationship allows staging only, so a compromised workflow can never publish a live version. A brand-new package cannot be staged, so publish its first version by hand.

```bash
npm trust github @dnd-mapp/<package> --repo dnd-mapp/<repo> --file release.yaml --env npm --allow-stage-publish
```

## Versioning

This repository is released with `vX.Y.Z` tags and GitHub Releases, like the packages. Its own [release workflow](.github/workflows/release.yaml) runs the action on its tags. Consumers pin a commit SHA, so a new release never changes a workflow until the pin is updated. Renaming or removing an input or output, or changing a default, is a breaking change.

## Changelog

Notable changes for consumers of this action are listed in the [changelog](CHANGELOG.md).

## Contributing

Contributions are welcome. See the [contributing guide](CONTRIBUTING.md) for details.

## License

[MIT](LICENSE) © D&D Mapp
