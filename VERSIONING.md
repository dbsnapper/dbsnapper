# Documentation Versioning Guide

This guide explains how to use the documentation versioning system powered by [Mike](https://github.com/jimporter/mike).

## Quick Start

### View Available Versions

```bash
source .venv/bin/activate
mike list
```

### Local Development

```bash
# Serve versioned docs locally with version switcher
mike serve --dev-addr=127.0.0.1:8000

# Serve regular docs (no version switcher)
mkdocs serve --dev-addr=127.0.0.1:8001
```

### Creating New Versions

```bash
# Deploy a new version (e.g., 2.8)
mike deploy --update-aliases 2.8 latest --title="2.8 (Latest)"

# Set as default version
mike set-default latest

# Deploy without updating "latest" alias
mike deploy 2.7 --title="2.7"
```

### Version Management

```bash
# List all versions
mike list

# Delete a version
mike delete 2.6

# Rename a version
mike retitle 2.7 "2.7 (Stable)"

# Deploy development version
mike deploy dev --title="Development"
```

## Automated Deployment

### GitHub Actions Integration

The documentation automatically deploys when:

1. **Tags are pushed** (e.g., `v2.8`):
   - Creates versioned documentation (`2.8`)
   - Updates `latest` alias to point to new version
   - Sets new version as default

2. **Main/master branch is pushed**:
   - Creates/updates `dev` version for testing

### Example Workflow

```bash
# Create and push a new version
git tag v2.8
git push origin v2.8

# This triggers CI to:
# - Extract version "2.8" from tag "v2.8"
# - Deploy as version "2.8" with "latest" alias
# - Set "2.8" as default version
```

## Version Structure

- **latest**: Always points to the most recent stable version
- **dev**: Latest development version from main/master branch
- **2.8, 2.7, etc.**: Specific version numbers from tags

## URLs

- `https://docs.dbsnapper.com/` - Default version (latest)
- `https://docs.dbsnapper.com/latest/` - Latest version
- `https://docs.dbsnapper.com/2.8/` - Specific version
- `https://docs.dbsnapper.com/dev/` - Development version

## Version Switcher

The version switcher appears in the top-right corner of the documentation site when multiple versions are available. Users can easily switch between versions without losing their place in the documentation.

## Troubleshooting

### Local Development Issues

```bash
# If mike serve fails, check versions
mike list

# If no versions exist, create one
mike deploy --update-aliases 2.7 latest --title="2.7 (Latest)"
mike set-default latest
```

### GitHub Actions Issues

- Ensure tags follow `v*` format (e.g., `v2.8`)
- Check that `fetch-depth: 0` is set in checkout action
- Verify git credentials are configured in workflow

### Version Conflicts

```bash
# Fix version conflicts by recreating
mike delete conflicted-version
mike deploy --update-aliases 2.8 latest --title="2.8 (Latest)"
```

For more details, see the [Mike documentation](https://github.com/jimporter/mike).