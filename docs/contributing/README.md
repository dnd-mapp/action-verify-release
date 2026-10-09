# Contributing to dnd-mapp/action-verify-release

This page adds the details of `dnd-mapp/action-verify-release` to the [shared contributing guide](https://github.com/dnd-mapp/.github/blob/main/CONTRIBUTING.md). Read that guide first.

This action decides whether a release of a D&D Mapp package may go ahead. A mistake here can block a release or let a wrong one through, so keep changes small and deliberate.

## Project layout

The action lives in `action.yaml` at the repository root, so consumers reference it as `dnd-mapp/action-verify-release`.

| Path                             | Purpose                                                                                            |
|:---------------------------------|:---------------------------------------------------------------------------------------------------|
| `action.yaml`                    | Checks the tag and the changelog, and writes the release notes                                     |
| `renovate.json`                  | The Renovate config of this repository, which extends the shared preset `dnd-mapp/config-renovate` |
| `.github/actions/ci/action.yaml` | The checks that the pull request, push, and release workflows run                                  |
| `.github/workflows/release.yaml` | Releases this repository, using the action on its own tags                                         |
| `.github/actionlint.yaml`        | Declares the `ubuntu-26.04` runner label, which actionlint does not know yet                       |

## Changing the action

Keep the action to its one purpose: every check that can fail for a fixable reason, run before any job with side effects. Staging the package and creating the GitHub Release are plain steps in the release workflow of each package, so they stay out of this action.

Pass inputs and outputs into `run` steps through `env`, and read them as shell variables. Never interpolate `${{ }}` expressions into a script, because a value with quotes or spaces would break or change the command.

actionlint does not read `action.yaml` itself. It checks the file through the release workflow of this repository, which runs the action on its own tags. Keep that usage in place when you change an input, so a rename is caught before a release.

When you add or change an input, output, or check, update these files in the same pull request.

- The `action.yaml` file, including the `description` of the input or output.
- The tables and the "What it checks" list in the README.

## Checks

This repository runs only the [shared checks](https://github.com/dnd-mapp/.github/blob/main/CONTRIBUTING.md#checks): `format-check`, `lint-md`, and actionlint.

## Changelog and versioning

This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Record every notable change for consumers under `[Unreleased]` in `CHANGELOG.md`, using the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

Release workflows depend on the names of the inputs and outputs and on the defaults. Renaming or removing one, changing a default, and adding a check that can fail an existing release are breaking changes for consumers. Say so in the changelog entry.

## Releasing

1. Run the [prepare release workflow](../../.github/workflows/prepare-release.yaml) on `main` with the part of the version to bump, for example `gh workflow run prepare-release.yaml -f bump=minor`. It opens the `chore: release X.Y.Z` pull request with auto-merge on.
2. Review and approve the pull request. Once it merges, the `tag` job of the [push workflow](../../.github/workflows/push-main.yaml) creates the annotated tag `vX.Y.Z` on the merge commit.
3. The [release workflow](../../.github/workflows/release.yaml) runs the CI checks, verifies the tag and the changelog with the action, and creates the GitHub Release, which opens a discussion in the Announcements category.
4. Update the SHA pins in the package repositories to the tagged commit.
