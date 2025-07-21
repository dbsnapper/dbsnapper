---
title: Quick Start Guide
description: Get started with DBSnapper in minutes - create your first sanitized database snapshot and load it into a development environment.
---

# Quick Start Guide

This comprehensive guide will take you from zero to your first database snapshot in under 15 minutes. You'll learn how to set up DBSnapper, create test databases, build snapshots, and troubleshoot common issues.

<!-- prettier-ignore-start -->
!!! success "What You'll Accomplish"

    By the end of this guide, you will have:
    
    - ✅ Set up test databases for learning DBSnapper
    - ✅ Configured the DBSnapper Agent
    - ✅ Created your first database snapshot
    - ✅ Loaded the snapshot into a development environment
    - ✅ Verified everything works correctly
<!-- prettier-ignore-end -->

## Prerequisites Check

<!-- prettier-ignore-start -->
!!! info "Before You Start"

    **Required:**
    
    - [DBSnapper Agent installed](installation.md) - Download and install the CLI tool
    - **15 minutes** - This guide is designed to be completed quickly
    - **Basic command line knowledge** - You'll run terminal/command prompt commands
    
    **Optional (we'll set up test databases for you):**
    
    - Existing PostgreSQL or MySQL database
    - Docker (recommended for isolated testing)
<!-- prettier-ignore-end -->

### Verify Installation

First, confirm DBSnapper is properly installed:

```bash
# Check DBSnapper version
dbsnapper --version

# Run configuration check
dbsnapper config check
```

If you see version information, you're ready to proceed. If not, return to the [Installation Guide](installation.md).

## Option A: Quick Start with Test Databases (Recommended)

This option creates everything you need for testing, including sample databases.

### Step 1: Set Up Test Databases with Docker

We'll create PostgreSQL containers with sample data for a complete learning experience:

```bash
# Create a Docker network for our test databases
docker network create dbsnapper-test 2>/dev/null || true

# Start source database with sample data
docker run -d \
  --name postgres-source \
  --network dbsnapper-test \
  -e POSTGRES_DB=sample_app \
  -e POSTGRES_USER=dbuser \
  -e POSTGRES_PASSWORD=dbpass123 \
  -p 5432:5432 \
  postgres:14

# Start destination database (initially empty)
docker run -d \
  --name postgres-dest \
  --network dbsnapper-test \
  -e POSTGRES_DB=dev_app \
  -e POSTGRES_USER=dbuser \
  -e POSTGRES_PASSWORD=dbpass123 \
  -p 5433:5432 \
  postgres:14

# Wait for databases to be ready
sleep 10
```

### Step 2: Add Sample Data to Source Database

```bash
# Create sample tables and data
docker exec -i postgres-source psql -U dbuser -d sample_app << 'EOF'
-- Create sample tables
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    ssn VARCHAR(11),  -- Sensitive data we'll sanitize later
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    amount DECIMAL(10,2),
    order_date TIMESTAMP DEFAULT NOW(),
    credit_card VARCHAR(20)  -- Sensitive data
);

-- Insert sample data
INSERT INTO users (email, first_name, last_name, phone, ssn) VALUES
    ('john.doe@example.com', 'John', 'Doe', '555-1234', '123-45-6789'),
    ('jane.smith@example.com', 'Jane', 'Smith', '555-5678', '987-65-4321'),
    ('bob.johnson@example.com', 'Bob', 'Johnson', '555-9012', '456-78-9012');

INSERT INTO orders (user_id, amount, credit_card) VALUES
    (1, 99.99, '4111111111111111'),
    (2, 149.50, '4222222222222222'),
    (1, 75.25, '4111111111111111'),
    (3, 200.00, '4333333333333333');

-- Show the data
SELECT 'Created ' || COUNT(*) || ' users' FROM users;
SELECT 'Created ' || COUNT(*) || ' orders' FROM orders;
EOF
```

### Step 3: Initialize DBSnapper Configuration

```bash
# Create initial configuration
dbsnapper config init

# Verify configuration was created
dbsnapper config check
```

<!-- prettier-ignore-start -->
!!! tip "What Happens During First Run"

    When you run `dbsnapper config init`, the following occurs:
    
    1. **Configuration Directory**: Creates `~/.config/dbsnapper/` directory
    2. **Configuration File**: Generates `dbsnapper.yml` with default settings
    3. **Secret Key**: Creates a unique secret key for encryption
    4. **Working Directory**: Sets up local storage for snapshots
    5. **Engine Detection**: Discovers available database tools (psql, mysql, docker)
<!-- prettier-ignore-end -->

### Step 4: Configure Your First Target

Edit the configuration file to add your test databases:

```bash
# Open configuration file in your default editor
# The file is located at ~/.config/dbsnapper/dbsnapper.yml
```

Add this target configuration:

```yaml
targets:
  quickstart_demo:
    snapshot:
      src_url: postgresql://dbuser:dbpass123@localhost:5432/sample_app?sslmode=disable
      dst_url: postgresql://dbuser:dbpass123@localhost:5433/dev_app?sslmode=disable
```

<!-- prettier-ignore-start -->
!!! warning "Understanding Target Configuration"

    **Source (`src_url`)**:
    - The database you want to snapshot (port 5432)
    - Contains your production-like data
    - DBSnapper only **reads** from this database
    
    **Destination (`dst_url`)**:
    - Where snapshots are loaded (port 5433)
    - Will be **completely replaced** when loading snapshots
    - **Never use a production database as destination**
<!-- prettier-ignore-end -->

### Step 5: Verify Target Configuration

```bash
# List and verify your targets
dbsnapper targets
```

Expected output showing your target with connection status:

```
╭──────────────────────────────────────────────────────────────────────────────╮
│                                   Targets                                    │
├──────────────────┬───────────────┬──────────────────┬─────────────┬─────────┤
│ Target           │ Source Status │ Dest Status      │ Source Size │ Engine  │
├──────────────────┼───────────────┼──────────────────┼─────────────┼─────────┤
│ quickstart_demo  │ ✅ Connected  │ ✅ Connected     │ 2.1 MB      │ pglocal │
└──────────────────┴───────────────┴──────────────────┴─────────────┴─────────┘
```

<!-- prettier-ignore-start -->
!!! error "Connection Troubleshooting"

    If you see **❌ Connection Failed**:
    
    **Check Database Status:**
    ```bash
    docker ps  # Verify containers are running
    docker logs postgres-source  # Check for errors
    ```
    
    **Test Connections Manually:**
    ```bash
    # Test source database
    docker exec postgres-source psql -U dbuser -d sample_app -c "SELECT version();"
    
    # Test destination database
    docker exec postgres-dest psql -U dbuser -d dev_app -c "SELECT version();"
    ```
    
    **Common Issues:**
    - **Port conflicts**: Another PostgreSQL running on port 5432/5433
    - **Container startup time**: Wait 30 seconds and try again
    - **Network issues**: Restart Docker and try again
<!-- prettier-ignore-end -->

### Step 6: Create Your First Snapshot

```bash
# Build a snapshot of your source database
dbsnapper build quickstart_demo
```

<!-- prettier-ignore-start -->
!!! info "What Happens During Snapshot Creation"

    **Phase 1: Database Connection**
    - Validates source database connectivity
    - Checks user permissions
    - Analyzes database schema and size
    
    **Phase 2: Data Export**
    - Uses `pg_dump` or `mysqldump` to create dump
    - Streams data to avoid large disk usage
    - Applies any configured sanitization rules
    
    **Phase 3: Storage**
    - Compresses the snapshot
    - Stores locally in working directory
    - Uploads to cloud storage (if configured)
    
    **Phase 4: Metadata Registration**
    - Records snapshot details
    - Indexes for easy retrieval
    - Updates target information
<!-- prettier-ignore-end -->

Expected output:
```
Building snapshot for target: quickstart_demo
✅ Connected to source database
✅ Analyzing database schema (4 tables found)
✅ Creating snapshot (2.1 MB)
✅ Snapshot created successfully
   - Snapshot ID: quickstart_demo_20240301_143022
   - Size: 2.1 MB
   - Duration: 12 seconds
```

### Step 7: Verify Snapshot Creation

```bash
# View all snapshots for your target
dbsnapper target quickstart_demo
```

You should see your snapshot listed with details:

```
╭──────────────────────────────────────────────────────────────────────────────╮
│                      Snapshots for quickstart_demo                          │
├─────┬─────────────────────────────┬────────────┬─────────────┬──────────────────┤
│ Idx │ Snapshot ID                 │ Created    │ Size        │ Status           │
├─────┼─────────────────────────────┼────────────┼─────────────┼──────────────────┤
│ 0   │ quickstart_demo_20240301... │ 2 min ago  │ 2.1 MB      │ ✅ Available     │
└─────┴─────────────────────────────┴────────────┴─────────────┴──────────────────┘
```

### Step 8: Load Snapshot into Development Database

```bash
# Load the most recent snapshot (index 0)
dbsnapper load quickstart_demo 0
```

<!-- prettier-ignore-start -->
!!! danger "Destructive Operation Warning"

    **The destination database will be completely replaced:**
    
    1. **Drops** the existing destination database
    2. **Creates** a new empty database
    3. **Restores** data from the snapshot
    4. **All existing data** in the destination is **permanently lost**
    
    **Always verify your destination URL before running load!**
<!-- prettier-ignore-end -->

Expected output:
```
Loading snapshot quickstart_demo_20240301_143022
⚠️  WARNING: This will completely replace the destination database
✅ Destination database dropped
✅ New database created
✅ Restoring data (4 tables)
✅ Snapshot loaded successfully
   - Tables restored: 4
   - Rows imported: 7
   - Duration: 8 seconds
```

### Step 9: Validate the Snapshot Load

Verify that your snapshot was loaded correctly:

```bash
# Check the destination database
docker exec postgres-dest psql -U dbuser -d dev_app -c "
SELECT 
    'users' as table_name, 
    COUNT(*) as row_count 
FROM users
UNION ALL
SELECT 
    'orders' as table_name, 
    COUNT(*) as row_count 
FROM orders;
"
```

You should see:
```
 table_name | row_count 
------------+-----------
 users      |         3
 orders     |         4
```

## Option B: Using Existing Databases

If you already have databases set up, follow these streamlined steps:

### Quick Configuration

1. **Initialize DBSnapper**:
   ```bash
   dbsnapper config init
   dbsnapper config check
   ```

2. **Add Your Target** (edit `~/.config/dbsnapper/dbsnapper.yml`):
   ```yaml
   targets:
     my_app:
       snapshot:
         src_url: postgresql://user:pass@localhost:5432/production_db
         dst_url: postgresql://user:pass@localhost:5432/development_db
   ```

3. **Verify and Build**:
   ```bash
   dbsnapper targets
   dbsnapper build my_app
   dbsnapper load my_app 0
   ```

## Validation Checklist

Confirm everything is working correctly:

<!-- prettier-ignore-start -->
!!! success "Validation Steps"

    **✅ Installation Verified**
    ```bash
    dbsnapper --version  # Shows version info
    ```
    
    **✅ Configuration Valid**
    ```bash
    dbsnapper config check  # All green checkmarks
    ```
    
    **✅ Target Connectivity**
    ```bash
    dbsnapper targets  # Shows "Connected" status
    ```
    
    **✅ Snapshot Created**
    ```bash
    dbsnapper target quickstart_demo  # Lists your snapshot
    ```
    
    **✅ Data Loaded Successfully**
    ```bash
    # Verify data in destination database
    docker exec postgres-dest psql -U dbuser -d dev_app -c "SELECT COUNT(*) FROM users;"
    ```
<!-- prettier-ignore-end -->

## Common Pitfalls and Solutions

### Database Connection Issues

<!-- prettier-ignore-start -->
!!! failure "Problem: 'Connection refused' errors"

    **Symptoms:**
    - `dbsnapper targets` shows ❌ Connection Failed
    - Error messages about refused connections
    
    **Solutions:**
    ```bash
    # 1. Check if database is running
    docker ps | grep postgres
    
    # 2. Verify port availability
    netstat -ln | grep 5432
    
    # 3. Test manual connection
    psql -h localhost -p 5432 -U dbuser -d sample_app
    
    # 4. Check firewall/network settings
    telnet localhost 5432
    ```
<!-- prettier-ignore-end -->

### Permission Problems

<!-- prettier-ignore-start -->
!!! failure "Problem: 'Permission denied' errors"

    **Symptoms:**
    - Can connect but cannot create dumps
    - Load operations fail with permission errors
    
    **Solutions:**
    ```bash
    # 1. Verify database user permissions
    docker exec postgres-source psql -U dbuser -d sample_app -c "
    SELECT 
        current_user as user, 
        session_user, 
        current_database() as database;
    "
    
    # 2. Grant necessary permissions (if using existing database)
    GRANT CONNECT ON DATABASE sample_app TO dbuser;
    GRANT USAGE ON SCHEMA public TO dbuser;
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO dbuser;
    ```
<!-- prettier-ignore-end -->

### Configuration Errors

<!-- prettier-ignore-start -->
!!! failure "Problem: 'Invalid configuration' errors"

    **Symptoms:**
    - YAML parsing errors
    - Missing required fields
    
    **Solutions:**
    ```bash
    # 1. Validate YAML syntax
    python -c "import yaml; yaml.safe_load(open('~/.config/dbsnapper/dbsnapper.yml'))"
    
    # 2. Use example configuration
    dbsnapper config init --force  # Regenerate with defaults
    
    # 3. Check indentation (use spaces, not tabs)
    cat -A ~/.config/dbsnapper/dbsnapper.yml
    ```
<!-- prettier-ignore-end -->

### Tool Dependencies

<!-- prettier-ignore-start -->
!!! failure "Problem: 'Database tools not found' errors"

    **Symptoms:**
    - `dbsnapper config check` shows missing tools
    - Build operations fail
    
    **Solutions:**
    ```bash
    # 1. Install PostgreSQL client tools
    # macOS
    brew install postgresql
    
    # Ubuntu/Debian
    sudo apt-get install postgresql-client
    
    # 2. Install MySQL client tools
    # macOS
    brew install mysql-client
    
    # Ubuntu/Debian
    sudo apt-get install mysql-client
    
    # 3. Alternative: Use Docker engines
    # Add to dbsnapper.yml:
    docker:
      images:
        postgres: postgres:14
        mysql: mysql:8.0
    ```
<!-- prettier-ignore-end -->

## Understanding What Happens Next

<!-- prettier-ignore-start -->
!!! tip "Snapshot Lifecycle Understanding"

    **Build Process:**
    1. DBSnapper connects to your source database
    2. Creates a logical dump using native tools
    3. Applies sanitization rules (if configured)
    4. Compresses and stores the snapshot
    5. Records metadata for tracking
    
    **Load Process:**
    1. DBSnapper connects to destination server
    2. Drops the destination database entirely
    3. Creates a fresh database with the same name
    4. Restores all data from the snapshot
    5. Rebuilds indexes and constraints
    
    **Key Points:**
    - Source database is never modified
    - Destination database is completely replaced
    - Snapshots are point-in-time copies
    - Multiple snapshots can be stored per target
<!-- prettier-ignore-end -->

## Your Next Steps

Congratulations! You've successfully created and loaded your first database snapshot. Here's what to explore next:

### 1. Add Data Sanitization (High Priority)

Learn to remove sensitive data from snapshots:

```yaml
# Add to your target configuration
sanitization:
  query_engine: postgres
  queries:
    - "UPDATE users SET ssn = 'XXX-XX-' || RIGHT(ssn, 4)"
    - "UPDATE orders SET credit_card = 'XXXX-XXXX-XXXX-' || RIGHT(credit_card, 4)"
```

**Next:** [Sanitization Guide](sanitize/introduction.md)

### 2. Set Up Team Sharing (Medium Priority)

Share snapshots securely with your team:

1. **[Create DBSnapper Cloud Account](https://app.dbsnapper.com/sign_up)** - Free trial available
2. **[Configure Storage Profiles](dbsnapper-cloud/storage_profiles.md)** - Use your cloud storage
3. **[Set Up SSO](dbsnapper-cloud/sso/sso-okta-oidc.md)** - Enterprise authentication

**Next:** [DBSnapper Cloud Guide](dbsnapper-cloud/introduction.md)

### 3. Automate with CI/CD (Medium Priority)

Integrate DBSnapper into your development workflow:

1. **GitHub Actions**: [Automated Snapshots](articles/dbsnapper-github-actions-ecs-simplified.md)
2. **VS Code Extension**: [In-Editor Management](https://marketplace.visualstudio.com/items?itemName=dbsnapper.vscode-dbsnapper)
3. **Terraform Provider**: [Infrastructure as Code](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest)

### 4. Optimize for Production (Lower Priority)

Advanced configuration for production use:

1. **[Data Subsetting](subset/introduction.md)** - Create smaller, focused snapshots
2. **[Performance Tuning](configuration.md#performance-optimization)** - Optimize for large databases
3. **[Storage Configuration](cloud-storage-engines/introduction.md)** - Set up cloud storage

## Clean Up Test Environment

When you're done experimenting, clean up the test containers:

```bash
# Stop and remove test containers
docker stop postgres-source postgres-dest
docker rm postgres-source postgres-dest

# Remove test network
docker network rm dbsnapper-test

# Optional: Remove test configuration
rm -rf ~/.config/dbsnapper
```

## Get Help and Support

**Community Resources:**
- [GitHub Discussions](https://github.com/dbsnapper/dbsnapper/discussions) - Community Q&A
- [GitHub Issues](https://github.com/dbsnapper/dbsnapper/issues) - Bug reports and feature requests
- [Discord Community](https://discord.gg/dbsnapper) - Real-time chat support

**Documentation:**
- [Configuration Reference](configuration.md) - Complete configuration options
- [Command Reference](cmd/dbsnapper.md) - Full CLI documentation
- [Database Engines](database-engines/introduction.md) - Database-specific guides

**Enterprise Support:**
- [Contact Sales](https://dbsnapper.com/contact) - Enterprise features and support
- [Professional Services](https://dbsnapper.com/services) - Implementation assistance

---

**Ready for the next level?** Choose your path from the next steps above, or dive deeper into [advanced configuration](configuration.md).