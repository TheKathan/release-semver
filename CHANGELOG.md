# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed
- **BREAKING**: Fixed workflow to correctly implement release-based versioning
- Action now creates official release tags (v1.0.0) when merging to release branches
- Prerelease tags (v1.0.0-alpha.1, v1.0.0-fix.1, etc.) only created when `create-prerelease: "true"`
- Adopted trunk-semver branch patterns and suffixes for consistency
- Workflow now only triggers on `main` and `release/**` branches (removed feature/fix/docs/etc triggers)

### Added
- Support for all trunk-semver branch types: feature, fix, chore, docs, refactor, perf, test, build, ci, style
- Each branch type gets appropriate suffix for prerelease tags (alpha, fix, chore, docs, etc.)
- Custom `prerelease-suffix` input for feature branches (default: "alpha")

### Removed
- Removed `enable-pr-tags` input (replaced with `create-prerelease`)
- Removed unnecessary workflow triggers on feature/fix/docs/etc branches
- Removed `pr-number` and `pr-target` outputs (no longer needed)

## [1.0.0] - 2026-02-13

### Added
- Complete GitHub Action for release-based semantic versioning
- Support for release branches (minor version bumps)
- Support for hotfix branches (patch version bumps)
- Optional prerelease tag creation for feature/fix merges to release branches
- Smart tag detection to prevent duplicate tags
- Support for custom major version and prerelease suffix
- Configurable branch patterns for custom workflows
- Outputs: `version`, `is-prerelease`, `branch`, `source-branch`, `previous-version`, `bump-type`
- Optional PR comments with version information
- Optional job summary with version details
- Comprehensive documentation with workflow examples
- GitHub workflows for testing and releases
- Branch protection rulesets for main, release, and hotfix branches
- Issue templates (bug reports and feature requests)
- Pull request template with checklist

### Features
- Release branches (`release/**`) merged to main create minor version bumps
- Hotfix branches (`hotfix/**`) merged to main create patch version bumps
- Feature branches (`feature/*`, `feat/*`) merged to release branches create minor prerelease bumps
- Fix branches (`fix/*`, `bugfix/*`) merged to release branches create patch prerelease bumps
- Automatic tag creation and push
- Major version tag updates on official releases (e.g., v1 → v1.2.0)
- Prerelease support with configurable suffix (alpha, beta, rc, etc.)

[Unreleased]: https://github.com/TheKathan/release-semver/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/TheKathan/release-semver/releases/tag/v1.0.0
