# Contributing

Thank you for your interest in contributing to `dnd-mapp/action-verify-release`.

This action decides whether a release of a D&D Mapp package may go ahead. A mistake here can block a release or let a wrong one through, so please keep changes small and deliberate.

## Before you start

Open an [issue](https://github.com/dnd-mapp/action-verify-release/issues) to discuss any change beyond a typo fix before you send a pull request. This avoids work on changes that do not fit the goals of the action.

## Development setup

The required Node and pnpm versions are set in `devEngines` in `package.json`. They are enforced through `engineStrict`, so installing with other versions fails.

Install the dependencies with:

```bash
pnpm install
```

Dependency versions live in the catalogs in `pnpm-workspace.yaml`, which uses `catalogMode: strict`. Add or bump versions there and reference them in `package.json`. Use `catalog:` for the default catalog and a named catalog such as `catalog:prettier` for a group of tools.

Newly published releases are held back for three days through `minimumReleaseAge`. You may need to wait before you can bump to a very recent version.

Install [actionlint](https://github.com/rhysd/actionlint) to lint the workflows locally, for example with `brew install actionlint`. CI runs the version that `.github/actions/ci/action.yaml` pins.

## Git hooks

[Lefthook](https://lefthook.dev/) installs the Git hooks when you run `pnpm install`. The hooks are defined in `lefthook.yaml`.

| Hook         | Runs                                  | On                        |
|:-------------|:--------------------------------------|:--------------------------|
| `pre-commit` | Prettier and markdownlint-cli2 checks | The staged files          |
| `commit-msg` | commitlint                            | The message of the commit |

The pre-commit hooks only check files. Run `pnpm run format` to fix formatting issues and stage the result.

## Project layout

The action lives in `action.yaml` at the repository root, so consumers reference it as `dnd-mapp/action-verify-release`.

| Path                             | Purpose                                                                                            |
|:---------------------------------|:---------------------------------------------------------------------------------------------------|
| `action.yaml`                    | Checks the tag and the changelog, and writes the release notes                                     |
| `renovate.json`                  | The Renovate config of this repository, which extends the shared preset `dnd-mapp/renovate-config` |
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

## Checking the repository

Check and format the repository with these commands. CI runs `format-check`, `lint-md`, and actionlint. Run them yourself before you open a pull request.

```bash
pnpm run format-check
pnpm run format
pnpm run lint-md
actionlint
```

## Changelog and versioning

This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Record every notable change for consumers under `[Unreleased]` in `CHANGELOG.md`, using the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

Release workflows depend on the names of the inputs and outputs and on the defaults. Renaming or removing one, changing a default, and adding a check that can fail an existing release are breaking changes for consumers. Say so in the changelog entry.

## Releasing

1. Open a pull request with a single `chore: release X.Y.Z` commit. It sets `version` in `package.json`, renames `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD`, adds a fresh `[Unreleased]`, and updates the link references.
2. Merge it, then create an annotated tag `vX.Y.Z` on the merge commit and push the tag.
3. The [release workflow](.github/workflows/release.yaml) runs the CI checks, verifies the tag and the changelog with the action, and creates the GitHub Release.
4. Update the SHA pins in the package repositories to the tagged commit.

## Code style

Follow the rules in `.editorconfig`.

- Use UTF-8 and LF line endings.
- Indent with 4 spaces, or 2 spaces in `package.json` and `pnpm-*.yaml`.
- End every file with a newline and trim trailing whitespace.

Follow these rules for prose, including Markdown files.

- Never hard wrap prose. Write each paragraph or list item on a single line.
- Use US spelling, for example "color" and "behavior".
- Keep every sentence at or under 40 words.
- Pretty print Markdown tables so the columns line up, with alignment markers on every separator line.

## Branches

Create a branch from `main` for each change. Name it `<type>/<short-description>` in lowercase with hyphens between words, for example `feat/compare-link-input` or `fix/shallow-fetch`.

Use the same types as for commits.

## Commits

Write commit messages that follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

```text
<type>(<optional scope>): <description>
```

Use one of these types.

| Type       | Use for                                              |
|:-----------|:-----------------------------------------------------|
| `feat`     | A new input, output, or check                        |
| `fix`      | A correction to existing behavior                    |
| `docs`     | Changes to documentation only                        |
| `refactor` | Changes that do not alter the behavior of the action |
| `ci`       | Changes to the workflows of this repository          |
| `build`    | Changes to dependencies or tooling                   |
| `chore`    | Other maintenance that does not fit above            |

Write the description in the imperative mood, such as "add the compare link input". Mark a breaking change with `!` after the type or scope, and add a `BREAKING CHANGE:` footer that explains what consumers must do.

## Pull requests

- Keep each pull request to one change.
- Link the issue it addresses.
- Update the changelog and README in the same pull request.
- Use a title that follows the commit convention.
- If you have write access, turn on auto-merge once the pull request is open, with `gh pr merge <number> --auto --merge` or the "Enable auto-merge" button. It then merges as soon as it is approved and the checks pass.
- If auto-merge is off, the author merges the pull request once it is approved and the checks pass. A maintainer merges pull requests opened by a contributor without write access.
- Renovate merges its own minor and patch pull requests once the checks pass. A maintainer approves a major update from Renovate and turns on auto-merge for it.
- Update the branch when it falls behind `main`, because auto-merge waits until the branch is up to date. The update dismisses the approval, so the pull request needs a new review.

## License

By contributing, you agree that your contributions are licensed under the [MIT license](LICENSE).
