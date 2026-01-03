# Law Innovation - Organization Workflows

This repository contains reusable GitHub Actions workflows and templates for the Law Innovation organization.

## Organization Templates

### Pull Request Template

**Path:** `.github/PULL_REQUEST_TEMPLATE.md`

Standardized PR template used across all repositories. Automatically applied when creating a pull request.

**Sections:**
- **Description** - Describe changes and include Jira ticket number in PR title
- **Was this deployed?** - Checklist for staging/production deployment status
- **How Has This Been Tested?** - Test coverage checklist (Manual, Unit, Integration, E2E)

This template is automatically picked up by GitHub for all repositories in the Law-Innovation organization.

## Available Reusable Workflows

### 1. Version and Tag

**Path:** `.github/workflows/version-and-tag.yml`

Enforces organizational standards for versioning and releases:
- **Automatic release type detection** - main branch = stable, feature branch = release candidate
- **Semantic versioning** - patch/minor/major
- **Tag creation** - Creates git tags and GitHub releases

**Usage:**
```yaml
jobs:
  version:
    uses: Law-Innovation/.github/.github/workflows/version-and-tag.yml@main
    with:
      tag_prefix: 'your-app@'  # e.g., analytics-pipeline@, consenter-cb@
      semver_type: ${{ github.event.inputs.semverType }}  # patch/minor/major
```

**Outputs:**
- `version`: Calculated version (e.g., 1.2.3 or 1.2.3-rc.0)
- `tag`: Full tag (e.g., analytics-pipeline@1.2.3)
- `is_prerelease`: Whether this is a prerelease (true/false)
- `release_type`: stable or release-candidate

## Organizational Standards

### Release & Deployment Pattern

**Branch-based release types (enforced):**
- `main` branch → ALWAYS stable release
- Feature/PR branch → ALWAYS release candidate

**Deployment pattern (enforced):**
- Stable releases → Deploy to BOTH staging AND production
- Release candidates → Deploy to staging ONLY

**Benefits:**
- ✅ No version staleness between environments
- ✅ No manual mistakes (branch determines release type)
- ✅ Simpler UX (only choose semver type)
- ✅ Clear intent (main = production-ready, feature = testing)

## Example: Full Release Workflow

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      semverType:
        description: semver type?
        required: true
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  version:
    uses: Law-Innovation/.github/.github/workflows/version-and-tag.yml@main
    with:
      tag_prefix: 'your-app@'
      semver_type: ${{ github.event.inputs.semverType }}

  test:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Add your test steps here

  deploy:
    needs: test
    strategy:
      matrix:
        # Stable (main) → both staging and production
        # RC (feature) → staging only
        environment: ${{ fromJSON(github.ref == 'refs/heads/main' && '["staging", "production"]' || '["staging"]') }}
    runs-on: ubuntu-latest
    environment: ${{ matrix.environment }}
    steps:
      - uses: actions/checkout@v4
      # Add your deployment steps here
```

## Repositories Using These Patterns

- ✅ `analytics-pipeline`
- ✅ `consenter-system` (consenter-cb, contextual-consent)
- More to come...

## How This Repository Works

This is a special `.github` repository for the Law-Innovation organization:

1. **Templates** - PR templates are automatically available to all org repositories
2. **Reusable Workflows** - Centralized workflows can be called from any repository
3. **Documentation** - Single source of truth for organizational standards

## Contributing

When adding new reusable workflows:
1. Follow the established patterns above
2. Document inputs/outputs clearly
3. Add examples to this README
4. Test in a real repository before merging
5. Use the PR template when submitting changes

When updating templates:
1. Consider impact across all repositories
2. Announce changes to the team
3. Update this README with any changes

## Date Established

2026-01-03
