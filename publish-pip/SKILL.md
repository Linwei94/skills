---
name: publish-pip
description: Publish a Python package to PyPI via GitHub Release. Use this skill whenever the user says "publish", "release", "bump version", "发布", "准备发布", "发个新版本", or wants to create a new version of their Python package. Also trigger when the user mentions PyPI, pip publishing, version bumping, or creating a GitHub release for a Python project. Even if they just say "发布一下" or "release it", use this skill.
---

# Publish Python Package via GitHub Release

This skill handles the full workflow of bumping a Python package version and publishing it via GitHub Release (which triggers a GitHub Action to upload to PyPI).

## Prerequisites

- The project must be a Python package with version defined somewhere (pyproject.toml, setup.py, __init__.py, etc.)
- The project must have a GitHub remote
- The project must have a GitHub Action that publishes to PyPI on release (if not, warn the user)

## Workflow

### Step 1: Discover current version

Search the project for version definitions. Common locations (check all, adapt to what exists):

- `pyproject.toml` → `version = "X.Y.Z"`
- `setup.py` → `version="X.Y.Z"`
- `setup.cfg` → `version = X.Y.Z`
- `__init__.py` or `_version.py` → `__version__ = "X.Y.Z"`
- `package.json` (rare for Python, but possible)

Use Grep to find all files containing version strings. Identify which files need to be updated — some projects define the version in one place, others in multiple.

Report findings to the user:

> Current version: `X.Y.Z` (found in `pyproject.toml`, `package/__init__.py`)

### Step 2: Propose new version

Based on the changes since the last release, suggest a version bump:

- **Patch** (X.Y.Z → X.Y.Z+1): Bug fixes, minor improvements, no API changes
- **Minor** (X.Y.Z → X.Y+1.0): New features, non-breaking changes
- **Major** (X.Y.Z → X+1.0.0): Breaking changes

Check `git log` since the last tag to understand what changed. Present the suggestion:

> Changes since last release:
> - feat: SSH auto-detection
> - feat: auto-kill stale process
> - fix: suppress Flask banner
>
> Suggested version: `X.Y.Z` (patch/minor/major). OK?

Wait for user confirmation. The user may specify a different version.

### Step 3: Update version in all locations

Edit every file that contains the version string. Make sure all locations stay in sync.

### Step 4: Commit and push

Stage only the version-bumped files. Commit with message:

```
Bump version to X.Y.Z
```

Push to the current branch (usually main).

### Step 5: Check for GitHub Action

Verify that a publish workflow exists:

```bash
ls .github/workflows/
```

Look for workflows triggered by release events. If none exist, warn the user that no auto-publish action was found and they may need to publish manually.

### Step 6: Generate release notes

Review commits since the last tag/release:

```bash
git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD --oneline
```

Group changes into categories and draft release notes:

```markdown
## What's New

- **Feature name**: Brief description
- **Another feature**: Brief description

## Bug Fixes

- Fixed X
```

Present the draft to the user for confirmation.

### Step 7: Create GitHub Release

```bash
gh release create vX.Y.Z --title "vX.Y.Z" --notes "..."
```

Use the confirmed release notes. The tag format should match the project's existing convention (check previous tags with `git tag -l` — some projects use `v1.0.0`, others use `1.0.0`).

### Step 8: Confirm

Report the release URL and remind the user to check that the GitHub Action completed successfully:

> Release created: https://github.com/user/repo/releases/tag/vX.Y.Z
> Check GitHub Actions to confirm PyPI publish succeeded.

## Key Principles

- Always confirm the new version number with the user before making changes
- Keep all version locations in sync
- Match the project's existing tag naming convention
- Generate meaningful release notes from git history
- Never force-push or amend commits during this process
