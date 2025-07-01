# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the official documentation repository for **DBSnapper**, a database snapshotting tool that creates sanitized, development-ready snapshots of production databases. The site is built with MkDocs Material theme and deployed to docs.dbsnapper.com.

### AI Context

1. Run `repomix` at the end of each major phase to pack the codebase into an AI-friendly format for better context preservation
2. Refer to the `repomix-output.xml` file for an AI-friendly format for better context preservation.

## Development Commands

### Local Development

```bash
# Install dependencies (run in CI/CD, no local requirements file)
pip install mkdocs-material mkdocs-git-revision-date-localized-plugin pillow cairosvg mkdocs-exclude-search-plugin

# Start local development server with live reload
mkdocs serve

# Build static site for deployment
mkdocs build

# Deploy to GitHub Pages (done automatically via CI/CD)
mkdocs gh-deploy --force
```

### No Testing Framework

This is a documentation-only project with no automated tests. Validation is done through:

- Local preview with `mkdocs serve`
- Manual link checking
- Visual review of rendered content

## Architecture

### Technology Stack

- **MkDocs** with Material theme for static site generation
- **GitHub Pages** for hosting at docs.dbsnapper.com
- **GitHub Actions** for automated deployment on push to main
- **Python-based** toolchain with markdown extensions

### Content Structure

```
docs/
├── index.md                    # Welcome page
├── how-it-works.md            # Architecture overview
├── installation.md            # Setup instructions
├── quick-start.md            # Getting started guide
├── configuration.md          # Configuration reference
├── snapshot/                 # Snapshot creation workflows
├── sanitize/                # Data sanitization guides
├── subset/                  # Data subsetting documentation
├── share/                   # Sharing functionality
├── dbsnapper-cloud/         # Cloud platform documentation
├── database-engines/        # PostgreSQL & MySQL guides
├── cloud-storage-engines/   # S3 & Cloudflare R2 integration
├── cmd/                     # CLI command reference
└── articles/                # Tutorials and use cases
```

### Key Configuration Files

- `mkdocs.yml` - Site configuration, navigation, theme settings
- `.github/workflows/ci.yml` - Automated deployment pipeline
- `.github/copilot-instructions.md` - Comprehensive writing guidelines
- `overrides/` - Custom theme modifications

## Content Guidelines

### Core Terminology

- **Target** = database connection configuration
- **Snapshot** = database backup/export
- **Sanitization** = removing/masking sensitive data
- **Subset** = reducing data volume
- **DBSnapper Cloud** = SaaS platform
- **DBSnapper Agent** = CLI tool (`dbsnapper` command)

### Writing Standards

- User-centric content for developers and DevOps engineers
- Include practical code examples (YAML configs, shell commands, SQL)
- Use Material theme admonitions for important information
- Progressive documentation from basic to advanced features
- Consistent file naming: lowercase with hyphens

### Code Example Formats

```yaml
# YAML for configuration examples
targets:
  my_target:
    database_type: postgres
    hostname: localhost
```

```bash
# Shell commands for CLI usage
dbsnapper build my_target
dbsnapper load my_target latest
```

### Adding New Documentation

1. Create markdown file in appropriate directory
2. Add navigation entry to `mkdocs.yml` under `nav:` section
3. Include frontmatter with title and description
4. Test locally with `mkdocs serve`
5. Verify navigation and links work correctly

### Phase Completion Protocol:

When completing major refactoring phases or significant milestones:

1. Update the `mkdocs.yml` file with new navigation structure
2. Ensure all links are valid and point to the correct files
3. Run `mkdocs serve` to preview changes
4. Run `repomix` to pack codebase for AI context preservation
5. Commit changes with a clear message indicating the phase completion

## Deployment

### Automated Pipeline

- Triggers on push to main/master branches
- Uses GitHub Actions with Ubuntu runner
- Installs Python dependencies and deploys via `mkdocs gh-deploy --force`
- No manual deployment needed

### Dependencies

No local dependency files (requirements.txt, package.json). Dependencies are installed directly in CI/CD:

- mkdocs-material
- mkdocs-git-revision-date-localized-plugin
- pillow
- cairosvg
- mkdocs-exclude-search-plugin

## Content Organization

The documentation covers the complete DBSnapper ecosystem:

- **DBSnapper Agent**: CLI tool for database operations
- **DBSnapper Cloud**: SaaS platform for team collaboration
- **VSCode Extension**: In-editor snapshot management
- **Terraform Provider**: Infrastructure as Code support
- **GitHub Actions**: CI/CD automation
- **SSO Integration**: Enterprise authentication (Okta OIDC)

Database support includes PostgreSQL and MySQL with cloud storage integration for Amazon S3 and Cloudflare R2.
