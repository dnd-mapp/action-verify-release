# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- The action. It asserts that the tagged commit is on the base branch, verifies the changelog with `changelog verify`, and writes the release notes with `changelog notes`. It outputs `version` and `notes-file`.

[Unreleased]: https://github.com/dnd-mapp/action-verify-release/commits/main
