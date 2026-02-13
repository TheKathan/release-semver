# Release Semver

[![License](https://img.shields.io/github/license/TheKathan/release-semver)](LICENSE)

Release-based semantic versioning for GitHub Actions. Perfect for teams that use release branches and hotfix workflows.

## How It Works

Branch-based versioning system:

1. **Release Branches → Main** - Creates minor version bumps
   - Merge `release/v1.2.0` to `main` → Tag: `v1.2.0`
   - Automatically updates major version tag (e.g., `v1`)

2. **Hotfix Branches → Main** - Creates patch version bumps
   - Merge `hotfix/critical-fix` to `main` → Tag: `v1.1.1`
   - Quick fixes for production issues

3. **Feature/Fix → Release** (Optional) - Creates prerelease tags
   - Merge `feature/auth` to `release/v1.2.0` → Tag: `v1.2.0-alpha.1`
   - Toggle with `create-prerelease` input

## Quick Start

```yaml
name: Release
on:
  push:
    branches:
      - main
      - 'release/**'

jobs:
  version:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: TheKathan/release-semver@v1
        with:
          major-version: "1"
          create-prerelease: "false"  # Set to "true" to enable prerelease tags
```

## Workflow Example

### Standard Release Flow

#### Step 1: Create Release Branch

```bash
git checkout main
git pull
git checkout -b release/v1.2.0
git push -u origin release/v1.2.0
```

#### Step 2: Develop Features

Create feature branches from the release branch:

```bash
git checkout -b feature/new-auth release/v1.2.0
# ... make changes ...
git commit -m "Add authentication"
git push -u origin feature/new-auth
gh pr create --base release/v1.2.0 --title "Add authentication"
```

**Optional:** Enable `create-prerelease: "true"` in the workflow to create `v1.2.0-alpha.1`, `v1.2.0-alpha.2`, etc., when features are merged to the release branch.

#### Step 3: Merge to Main (Official Release)

When ready to release:

```bash
gh pr create --base main --head release/v1.2.0 --title "Release v1.2.0"
gh pr merge <pr-number> --merge
```

The action will:
- Detect merge from `release/v1.2.0` to `main`
- Create tag: `v1.2.0` (minor bump)
- Update major tag: `v1` → `v1.2.0`
- Create GitHub Release with auto-generated notes

### Hotfix Flow

For urgent production fixes:

```bash
git checkout main
git pull
git checkout -b hotfix/critical-bug
# ... make fixes ...
git commit -m "Fix critical bug"
git push -u origin hotfix/critical-bug
gh pr create --base main --title "Hotfix: Critical bug"
gh pr merge <pr-number> --merge
```

The action will:
- Detect merge from `hotfix/critical-bug` to `main`
- Create tag: `v1.1.1` (patch bump)
- Update major tag: `v1` → `v1.1.1`
- Create GitHub Release

## Branch Patterns

### Main Branch Merges

| Source Branch | Version Bump | Example |
|---------------|--------------|---------|
| `release/**` | Minor (1.1.0 → 1.2.0) | Merge `release/v1.2.0` → `v1.2.0` |
| `hotfix/**` | Patch (1.1.0 → 1.1.1) | Merge `hotfix/fix-bug` → `v1.1.1` |

### Release Branch Merges (Prerelease)

When `create-prerelease: "true"`:

| Source Branch | Version Bump | Example |
|---------------|--------------|---------|
| `feature/*`, `feat/*` | Minor (1.1.0 → 1.2.0-alpha.1) | Merge `feature/auth` → `v1.2.0-alpha.1` |
| `fix/*`, `bugfix/*` | Patch (1.1.0 → 1.1.1-alpha.1) | Merge `fix/typo` → `v1.1.1-alpha.1` |
| Everything else | Patch (1.1.0 → 1.1.1-alpha.1) | Default behavior |

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `major-version` | `1` | Major version number |
| `prerelease-suffix` | `alpha` | Suffix for prerelease tags (e.g., `v1.2.0-alpha.1`) |
| `create-prerelease` | `false` | Create prerelease tags when merging to release branches |
| `github-token` | `${{ github.token }}` | Token for API access |
| `branch-patterns` | `""` | Custom branch patterns (JSON, optional) |
| `comment-on-pr` | `false` | Add version comment to PRs |
| `add-to-summary` | `false` | Add version to job summary |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The version tag created (e.g., `v1.2.0-alpha.1` or `v1.2.0`) |
| `is-prerelease` | `true` for prerelease tags, `false` for official releases |
| `branch` | Current branch name (main or release/**) |
| `source-branch` | Source branch from merged PR (release/**, hotfix/**, feature/**, etc.) |
| `previous-version` | Previous version tag |
| `bump-type` | Type of version bump (Minor or Patch) |

## Advanced Usage

### Enable Prerelease Tags

Enable automatic prerelease tags when merging to release branches:

```yaml
- uses: TheKathan/release-semver@v1
  with:
    create-prerelease: "true"
    prerelease-suffix: "beta"  # Creates v1.2.0-beta.1 instead of v1.2.0-alpha.1
```

### Custom Branch Patterns

Customize which branch patterns trigger minor vs patch bumps:

```yaml
- uses: TheKathan/release-semver@v1
  with:
    branch-patterns: |
      {
        "minor": ["feature/*", "feat/*", "enhancement/*"],
        "patch": ["fix/*", "bugfix/*", "hotfix/*", "chore/*"]
      }
```

### PR Comments & Job Summaries

```yaml
- uses: TheKathan/release-semver@v1
  with:
    comment-on-pr: "true"
    add-to-summary: "true"
```

Displays version information:

```
## 📦 Version: v1.2.0-alpha.1

| Field | Value |
|-------|-------|
| Type | Prerelease |
| Branch | release/v1.2.0 |
| Source | feature/new-auth |
| Bump | Minor |
| Previous | v1.1.0 |
```

### Using Version in Subsequent Steps

```yaml
- name: Version
  id: version
  uses: TheKathan/release-semver@v1

- name: Create Release
  if: steps.version.outputs.is-prerelease == 'false'
  env:
    GH_TOKEN: ${{ github.token }}
  run: gh release create ${{ steps.version.outputs.version }} --generate-notes

- name: Deploy
  if: steps.version.outputs.is-prerelease == 'false'
  run: |
    echo "Deploying version ${{ steps.version.outputs.version }}"
    # Your deployment commands here
```

## Comparison with Trunk Semver

| Feature | Release Semver | [Trunk Semver](https://github.com/TheKathan/trunk-semver) |
|---------|----------------|-----------------------------------------------------------|
| **Workflow** | Release branches + hotfixes | Direct to main |
| **Branching** | release/** → main, hotfix/** → main | feature/** → main |
| **Prereleases** | Optional on release branches | Optional on main |
| **Version Control** | Explicit release branches | Automatic on every merge |
| **Hotfixes** | ✅ Separate hotfix workflow | Manual tag creation |
| **Best For** | Teams with release cycles and staging | Continuous deployment to main |

## FAQ

### When should I use Release Semver vs Trunk Semver?

**Use Release Semver when:**
- You have release branches and staging environments
- You need to accumulate multiple features before releasing
- You need a separate hotfix workflow
- You want explicit control over when releases happen
- You follow Git Flow or similar branching strategies

**Use Trunk Semver when:**
- Every merge to main should create a version immediately
- You practice continuous deployment from main
- You don't use long-lived release branches
- You prefer trunk-based development

### How do I change the major version?

Update the `major-version` input:

```yaml
- uses: TheKathan/release-semver@v1
  with:
    major-version: "2"  # Creates v2.x.x tags
```

### Can I customize branch patterns?

Yes! Use the `branch-patterns` input to control which branch patterns trigger minor vs patch bumps for prereleases:

```yaml
- uses: TheKathan/release-semver@v1
  with:
    branch-patterns: |
      {
        "minor": ["feature/*", "feat/*", "enhancement/*"],
        "patch": ["fix/*", "hotfix/*", "bugfix/*", "chore/*"]
      }
```

**Note:** For main branch merges, only `release/**` and `hotfix/**` branches are allowed as sources.

### What happens if I merge other branches directly to main?

The action will fail with an error. Only `release/**` and `hotfix/**` branches can be merged to main:
- `release/**` → Creates minor version bump
- `hotfix/**` → Creates patch version bump

This ensures consistent versioning and prevents accidental releases.

### How do I enable prerelease tags?

Set `create-prerelease: "true"` in your workflow:

```yaml
- uses: TheKathan/release-semver@v1
  with:
    create-prerelease: "true"
    prerelease-suffix: "alpha"  # or "beta", "rc", etc.
```

This creates tags like `v1.2.0-alpha.1`, `v1.2.0-alpha.2`, etc., when merging to release branches.

## License

GPL-3.0

## Contributing

Found a bug or have an idea? Open an issue or submit a PR at [TheKathan/release-semver](https://github.com/TheKathan/release-semver)
