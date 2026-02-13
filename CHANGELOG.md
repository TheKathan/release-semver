# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Release-based semantic versioning workflow
- Release candidate creation on manual GitHub Release events (v1.2.3-re.N)
- Official release creation on push to main (v1.2.3)
- Smart tag detection to prevent duplicate tags
- Support for custom major version and release suffix
- Outputs: `version`, `is-release-candidate`, `branch`, `previous-version`, `bump-type`
- Optional PR comments with version information
- Optional job summary with version details

### Features
- Feature branches (`feature/*`, `feat/*`) bump minor version
- All other branches bump patch version
- Automatic tag creation and push
- Major version tag updates on official releases

[Unreleased]: https://github.com/TheKathan/release-semver/compare/v1.0.0...HEAD
