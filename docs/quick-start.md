---
title: Quick Start Guide
description: Get started with DBSnapper in minutes - choose your path based on what you want to accomplish.
---

# Quick Start Guide

Get DBSnapper running in minutes with the approach that fits your needs. Whether you're exploring DBSnapper for the first time or setting up your team's database snapshot workflow, we've got you covered.

## Choose Your Path

=== "🚀 Try DBSnapper (5 minutes)"

    **Perfect for:** First-time users who want to see DBSnapper in action quickly.

    **What you'll accomplish:**
    
    - ✅ Run DBSnapper in a container (no installation needed)
    - ✅ Create your first database snapshot
    - ✅ Load a snapshot into a development database
    
    **Prerequisites:** Docker installed and a test database available

    ### Step 1: Run DBSnapper with Docker

    Use our pre-built container with all tools included:

    ```bash
    docker run -it --rm ghcr.io/dbsnapper/dbsnapper:latest /bin/bash
    ```

    ### Step 2: Initialize Configuration

    ```bash
    dbsnapper config init
    ```

    ### Step 3: Configure a Test Target

    Edit the generated config to add your database:

    ```bash
    # Edit ~/.config/dbsnapper/dbsnapper.yml
    targets:
      my_test:
        snapshot:
          src_url: "postgresql://user:pass@host:5432/source_db"
          dst_url: "postgresql://user:pass@host:5432/test_db"
    ```

    ### Step 4: Create Your First Snapshot

    ```bash
    dbsnapper build my_test
    ```

    ### Step 5: Load the Snapshot

    ```bash
    dbsnapper load my_test 0
    ```

    🎉 **Success!** You've created and loaded your first snapshot. Ready to explore more features?

=== "⚙️ Set Up for Your Team"

    **Perfect for:** Teams who want to share snapshots and use cloud storage.

    **What you'll accomplish:**
    
    - ✅ Set up DBSnapper Cloud integration
    - ✅ Configure team storage profiles
    - ✅ Create shareable, sanitized snapshots
    - ✅ Enable SSO-based team access
    
    **Prerequisites:** [DBSnapper Cloud account](https://app.dbsnapper.com/sign_up) and cloud storage access

    ### Step 1: Install DBSnapper

    Choose your preferred installation method:

    ```bash
    # macOS with Homebrew
    brew install dbsnapper/tap/dbsnapper
    
    # Or use Docker
    docker pull ghcr.io/dbsnapper/dbsnapper:latest
    ```

    ### Step 2: Connect to DBSnapper Cloud

    ```bash
    dbsnapper config init
    dbsnapper auth token YOUR_AUTH_TOKEN
    ```

    ### Step 3: Configure Team Storage

    Add your team's storage profile in the cloud dashboard or config:

    ```yaml
    storage_profiles:
      team_storage:
        provider: s3
        bucket: your-team-snapshots
        awscli_profile: production
    ```

    ### Step 4: Create Team Targets

    Configure targets with sanitization and sharing:

    ```yaml
    targets:
      production_app:
        snapshot:
          src_url: "postgresql://user:pass@prod:5432/app"
          dst_url: "postgresql://user:pass@dev:5432/app_dev"
        storage_profile: team_storage
        sanitize:
          query_file: "sanitize.sql"
        # Enable team sharing via SSO groups
        sso_groups: ["developers", "qa-team"]
    ```

    ### Step 5: Build and Share Snapshots

    ```bash
    # Build original and sanitized snapshots
    dbsnapper sanitize production_app -n
    
    # Team members can now access via shared targets
    dbsnapper targets --shared
    ```

    🎉 **Success!** Your team can now access sanitized snapshots securely.

=== "🔧 Integrate with Your Workflow"

    **Perfect for:** Developers who want DBSnapper integrated into their daily workflow.

    **What you'll accomplish:**
    
    - ✅ Set up VSCode extension for in-editor access
    - ✅ Configure GitHub Actions for automated snapshots
    - ✅ Use Terraform provider for infrastructure as code
    - ✅ Enable MCP server for AI assistant integration
    
    **Prerequisites:** DBSnapper installed, VSCode/GitHub/Terraform as needed

    ### Step 1: Install VSCode Extension

    Install the [DBSnapper Extension](https://marketplace.visualstudio.com/items?itemName=dbsnapper.vscode-dbsnapper) from the VS Code Marketplace.

    Configure your workspace settings:

    ```json
    {
        "dbsnapper.configPath": "~/.config/dbsnapper/dbsnapper.yml",
        "dbsnapper.autoRefresh": true
    }
    ```

    ### Step 2: Set Up GitHub Actions

    Add the DBSnapper GitHub Action to your workflow:

    ```yaml
    # .github/workflows/db-snapshots.yml
    name: Database Snapshots
    on: 
      schedule:
        - cron: '0 2 * * *'  # Daily at 2 AM
    
    jobs:
      snapshot:
        runs-on: ubuntu-latest
        steps:
          - uses: dbsnapper/install-dbsnapper-agent-action@v1
            with:
              version: latest
          - name: Create Snapshot
            env:
              DBSNAPPER_SECRET_KEY: ${{ secrets.DBSNAPPER_SECRET_KEY }}
              DBSNAPPER_AUTHTOKEN: ${{ secrets.DBSNAPPER_AUTHTOKEN }}
            run: dbsnapper build production-db
    ```

    ### Step 3: Use Terraform Provider

    Manage DBSnapper resources with Infrastructure as Code:

    ```hcl
    terraform {
      required_providers {
        dbsnapper = {
          source = "dbsnapper/dbsnapper"
        }
      }
    }
    
    resource "dbsnapper_target" "production" {
      name = "production-api"
      snapshot = {
        src_url = var.database_url
        dst_url = var.dev_database_url
      }
      storage_profile = "team-s3"
    }
    ```

    ### Step 4: Enable MCP Server (Optional)

    For AI assistant integration:

    ```bash
    # Start MCP server
    dbsnapper mcp
    
    # Configure your AI assistant to use the MCP server
    # Server runs on default port for Claude Desktop/other clients
    ```

    🎉 **Success!** DBSnapper is now integrated into your development workflow.

## What's Next?

Now that you have DBSnapper running, explore these powerful features:

### 🛡️ **Data Sanitization**
Remove sensitive data from your snapshots:
- **[Sanitization Guide](sanitize/introduction.md)** - Learn sanitization strategies
- **[Configuration Examples](sanitize/configuration.md)** - Set up automated data cleaning

### 📊 **Database Subsetting**
Create smaller, focused snapshots:
- **[Subset Introduction](subset/introduction.md)** - Understand subsetting concepts
- **[Configuration Guide](subset/configuration.md)** - Set up intelligent data filtering

### ☁️ **Team Collaboration**
Share snapshots securely with your team:
- **[DBSnapper Cloud](dbsnapper-cloud/introduction.md)** - Central snapshot management
- **[SSO Setup](dbsnapper-cloud/sso/index.md)** - Configure team authentication

### ⚡ **Performance Optimization**
Optimize for your workload:
- **[CPU Configuration](configuration.md#performance-configuration)** - Multi-core snapshot processing
- **[Storage Profiles](dbsnapper-cloud/storage_profiles.md)** - Efficient cloud storage

## Need Help?

### Common Issues

**"Connection failed"** - Check your database URLs and network access

**"Tools not found"** - Install database client tools or use Docker mode:
```bash
# Check what's available
dbsnapper config check

# Use Docker-based tools
docker run -it ghcr.io/dbsnapper/dbsnapper:latest
```

**"Permission denied"** - Ensure database user has required permissions:
- `SELECT` access on source database  
- `CREATE DATABASE` access on destination server

### Get Support

- **[Configuration Reference](configuration.md)** - Complete configuration options
- **[Command Reference](cmd/dbsnapper.md)** - Full CLI documentation  
- **[GitHub Issues](https://github.com/dbsnapper/dbsnapper/issues)** - Report bugs or request features
- **[Community Discussions](https://github.com/dbsnapper/dbsnapper/discussions)** - Get help from the community

---

**Ready to dive deeper?** Check out our **[comprehensive configuration guide](configuration.md)** for advanced features and customization options.