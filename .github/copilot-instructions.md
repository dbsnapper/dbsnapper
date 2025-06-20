# GitHub Copilot Instructions for DBSnapper Documentation

This repository contains the MkDocs-based documentation for DBSnapper, a database snapshotting tool that creates sanitized development snapshots of production databases.

## Project Overview

**DBSnapper** is a tool that revolutionizes database snapshotting by bringing de-identified production data into development and testing workflows. The documentation covers:

- Database snapshot creation, sanitization, and loading
- Support for PostgreSQL and MySQL databases
- Cloud storage integration (Amazon S3, Cloudflare R2)
- DBSnapper Cloud platform features
- Command-line interface documentation
- Integration guides and tutorials

## Technology Stack

- **Documentation Generator**: MkDocs with Material theme
- **Markup**: Markdown with extensions (PyMdown, Admonition, etc.)
- **Hosting**: GitHub Pages (docs.dbsnapper.com)
- **Styling**: Material Design with custom CSS
- **Features**: Search, dark/light mode, navigation, social cards

## Content Structure & Guidelines

### Documentation Organization

```
docs/
├── index.md                    # Welcome page and overview
├── how-it-works.md            # Architecture and concepts
├── installation.md            # Setup instructions
├── quick-start.md            # Getting started guide
├── configuration.md          # Main configuration reference
├── snapshot/                 # Snapshot creation docs
├── sanitize/                # Data sanitization docs
├── subset/                  # Data subsetting docs
├── share/                   # Sharing functionality
├── dbsnapper-cloud/         # Cloud platform docs
├── database-engines/        # Database-specific guides
├── cloud-storage-engines/   # Storage provider guides
├── cmd/                     # CLI command reference
└── articles/                # Tutorials and guides
```

### Writing Style Guidelines

1. **User-Centric**: Write from the perspective of developers and DevOps engineers
2. **Practical**: Include code examples, configuration snippets, and screenshots
3. **Progressive**: Start with basic concepts, build to advanced features
4. **Consistent**: Use consistent terminology (e.g., "target" for database connection, "snapshot" for database backup)
5. **Clear**: Use simple, direct language; avoid jargon when possible

### Content Types

- **Guides**: Step-by-step instructions with examples
- **Reference**: Configuration options, CLI commands, API docs
- **Tutorials**: End-to-end workflows and use cases
- **Concepts**: Architecture explanations and theory

### Code Examples & Configuration

When adding code examples:

```yaml
# Use YAML for configuration examples
targets:
  my_target:
    database_type: postgres
    hostname: localhost
    port: 5432
```

```bash
# Use shell for CLI commands
dbsnapper build my_target
dbsnapper load my_target latest
```

```sql
-- Use SQL for database queries
SELECT * FROM users WHERE created_at > '2024-01-01';
```

### Admonitions

Use Material theme admonitions for important information:

```markdown
!!! note "Version Information"

This feature is available in DBSnapper v2.0+

!!! warning "Security Note"

Always review sanitization rules before loading production data

!!! tip "Pro Tip"

Use subset configurations to reduce snapshot size
```

Always put a blank line after the !!! admonition block to ensure proper rendering.

## File Naming Conventions

- Use lowercase with hyphens: `backup-amazon-rds-with-dbsnapper.md`
- Be descriptive: `dbsnapper-github-actions-ecs-simplified.md`
- Group related content in folders: `database-engines/postgresql-local.md`

## MkDocs Configuration

The site uses these key features:

- **Theme**: Material with custom logo and colors
- **Navigation**: Hierarchical menu structure defined in `mkdocs.yml`
- **Plugins**: Git revision dates, search, social cards
- **Extensions**: Code highlighting, tabs, task lists, emoji support

## Image and Asset Guidelines

- Store images in `docs/static/` directory
- Use descriptive filenames: `amazon-rds-postgres-target-screenshot.png`
- Include alt text for accessibility
- Optimize file sizes for web delivery
- Use consistent screenshot styling (browser chrome, consistent UI state)

## Common Tasks

### Adding New Documentation

1. Create the markdown file in the appropriate directory
2. Add navigation entry to `mkdocs.yml` under the `nav:` section
3. Include frontmatter with title and description:
   ```yaml
   ---
   title: Page Title
   description: Brief description for SEO and social sharing
   ---
   ```

### CLI Documentation

- Command reference files follow pattern: `cmd/dbsnapper_command.md`
- Include usage examples, common flags, and troubleshooting
- Auto-generated from CLI help text when possible

### Adding Code Examples

- Prefer complete, runnable examples
- Include context and explanation
- Show both input and expected output
- Use syntax highlighting for better readability

## Brand and Terminology

- **Product Name**: DBSnapper (not DB Snapper or dbsnapper in prose)
- **Core Concepts**:
  - Target = database connection configuration
  - Snapshot = database backup/export
  - Sanitization = removing/masking sensitive data
  - Subset = reducing data volume
- **Cloud Service**: DBSnapper Cloud
- **CLI Tool**: `dbsnapper` command

## Testing and Quality

Before submitting documentation:

1. **Local Testing**: Run `mkdocs serve` to preview changes
2. **Link Checking**: Verify all internal and external links work
3. **Code Examples**: Test all code snippets and configurations
4. **Screenshots**: Ensure images are current and properly sized
5. **Navigation**: Verify new pages appear in the navigation menu

## Development Workflow

When updating documentation:

1. Create feature branch from main
2. Make changes and test locally with `mkdocs serve`
3. Commit with descriptive messages
4. Submit pull request with documentation preview
5. Deploy automatically triggers on merge to main

This documentation serves developers, DevOps engineers, and database administrators who need to create safe, sanitized database snapshots for development and testing environments.
