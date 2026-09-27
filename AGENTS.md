# Agent instructions

## Project

This repository is the composite GitHub Action `dnd-mapp/action-verify-release`. The action in `action.yaml` checks a pushed release tag against `package.json` and `CHANGELOG.md`, and writes the release notes with the `changelog` bin from `@dnd-mapp/changelog-tools`.

- Keep the action to its checks. Staging the package and creating the GitHub Release are plain steps in the release workflow of each package.
- Run `format-check`, `lint-md`, and `actionlint` before you commit.
