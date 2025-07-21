---
title: Build and Load Snapshots
description: Complete guide to creating, managing, and restoring database snapshots with practical examples and best practices.
---

# Build and Load Snapshots

This guide covers everything you need to know about creating and loading database snapshots, from basic operations to advanced workflows and troubleshooting.

<!-- prettier-ignore-start -->
!!! tip "Quick Reference"

    **Create a snapshot**: `dbsnapper build <target>`  
    **Load latest snapshot**: `dbsnapper load <target> 0`  
    **List snapshots**: `dbsnapper target <target>`  
    **Load specific snapshot**: `dbsnapper load <target> <index>`
<!-- prettier-ignore-end -->

## Understanding Snapshots

A DBSnapper snapshot is a point-in-time copy of your database that includes:

- **Complete schema**: Tables, indexes, constraints, functions, views
- **All data**: Row-level data from all tables
- **Metadata**: Creation timestamp, size, target information
- **Compression**: Optimized storage using standard compression

### Snapshot Lifecycle

```mermaid
graph LR
    DB[(Source Database)] -->|build| Snap[Snapshot File]
    Snap -->|load| DevDB[(Development DB)]
    Snap -->|share| Cloud[Cloud Storage]
    Cloud -->|pull| Team[Team Member]
    Team -->|load| LocalDB[(Local DB)]
```

## Basic Build and Load Operations

### Simple Snapshot Creation

Given this basic target configuration:

```yaml title="~/.config/dbsnapper/dbsnapper.yml"
targets:
  my_app:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/development
```

Create your first snapshot:

```bash
# Create a snapshot of the production database
dbsnapper build my_app
```

**Expected Output:**
```
DBSnapper Agent - Version: 2.0.3
DBSnapper Cloud: Standalone Mode

START: Build Snapshot for target: my_app with engine: pglocal
--> Connecting to source database: postgresql://postgres:***@localhost:5432/production
--> Database connection successful (PostgreSQL 16.1)
--> Analyzing database schema (47 tables, 312 indexes)
--> Creating snapshot using pg_dump
--> Compressing snapshot (original: 245MB, compressed: 58MB)
--> Snapshot saved: ~/.dbsnapper/20240301_143022_my_app.zip
FINISH: Building DB Snapshot for target: my_app (Duration: 2m 34s)
```

### Loading a Snapshot

```bash
# Load the most recent snapshot (index 0)
dbsnapper load my_app 0
```

**Expected Output:**
```
START: Loading Snapshot #0 for target: my_app
--> Snapshot: 20240301_143022_my_app.zip (58MB)
--> Destination: postgresql://postgres:***@localhost:5432/development
--> Extracting snapshot to temporary directory
--> Dropping database 'development'
--> Creating database 'development'
--> Restoring schema (47 tables)
--> Restoring data (1.2M rows)
--> Rebuilding indexes and constraints
FINISH: Loading Snapshot for target: my_app (Duration: 1m 18s)
```

<!-- prettier-ignore-start -->
!!! warning "Destructive Operation"

    **The destination database will be completely dropped and recreated.**
    
    - All existing data in the destination database will be permanently lost
    - Always verify your `dst_url` before running `load`
    - Consider backing up important development work first
<!-- prettier-ignore-end -->

### Viewing Snapshots

List all snapshots for a target:

```bash
dbsnapper target my_app
```

**Output:**
```
╭─────────────────────────────────────────────────────────────────────────────╮
│                        Snapshots for my_app                                │
├─────┬─────────────────────────────┬────────────┬─────────────┬──────────────┤
│ Idx │ Snapshot ID                 │ Created    │ Size        │ Status       │
├─────┼─────────────────────────────┼────────────┼─────────────┼──────────────┤
│ 0   │ 20240301_143022_my_app      │ 2 min ago  │ 58.2 MB     │ ✅ Available │
│ 1   │ 20240301_120015_my_app      │ 3 hrs ago  │ 57.8 MB     │ ✅ Available │
│ 2   │ 20240228_183045_my_app      │ 1 day ago  │ 56.9 MB     │ ✅ Available │
└─────┴─────────────────────────────┴────────────┴─────────────┴──────────────┘
```

## Practical Examples

### E-commerce Application Snapshot

```yaml title="E-commerce target configuration"
targets:
  ecommerce_prod:
    snapshot:
      # Production database (read-only access)
      src_url: postgresql://readonly:{{`PROD_DB_PASS` | env}}@prod.example.com:5432/ecommerce
      # Local development database
      dst_url: postgresql://postgres:postgres@localhost:5432/ecommerce_dev
```

**Build with custom naming:**
```bash
# Create snapshot with descriptive tag
dbsnapper build ecommerce_prod --tag "pre-sale-deployment"
```

**Load for specific feature development:**
```bash
# Load into feature-specific database
dbsnapper load ecommerce_prod 0 --dst-override "postgresql://postgres:postgres@localhost:5432/feature_checkout"
```

### Multi-Database Application

```yaml title="Microservices snapshot configuration"
targets:
  user_service:
    snapshot:
      src_url: postgresql://app:password@prod-users.example.com:5432/users
      dst_url: postgresql://postgres:postgres@localhost:5432/users_dev
  
  order_service:
    snapshot:
      src_url: postgresql://app:password@prod-orders.example.com:5432/orders
      dst_url: postgresql://postgres:postgres@localhost:5432/orders_dev
  
  inventory_service:
    snapshot:
      src_url: mysql://app:password@prod-inventory.example.com:3306/inventory
      dst_url: mysql://root:root@localhost:3306/inventory_dev
```

**Build all services:**
```bash
# Create snapshots for all microservices
for service in user_service order_service inventory_service; do
    echo "Building snapshot for $service..."
    dbsnapper build $service --tag "$(date +%Y%m%d)-full-stack"
done
```

**Load complete development environment:**
```bash
# Load all services for local development
dbsnapper load user_service 0 &
dbsnapper load order_service 0 &
dbsnapper load inventory_service 0 &
wait
echo "Development environment ready!"
```

### Staging Environment Refresh

```yaml title="Staging refresh workflow"
targets:
  staging_refresh:
    snapshot:
      # Production database (sanitized copy)
      src_url: postgresql://readonly:password@prod.example.com:5432/app_sanitized
      # Staging database
      dst_url: postgresql://staging:password@staging.example.com:5432/app_staging
```

**Automated staging refresh:**
```bash
#!/bin/bash
# staging-refresh.sh

echo "Starting staging environment refresh..."

# Create fresh snapshot from sanitized production
dbsnapper build staging_refresh --tag "staging-refresh-$(date +%Y%m%d)"

# Load into staging environment
dbsnapper load staging_refresh 0

# Verify staging environment
if dbsnapper targets staging_refresh | grep -q "Connected"; then
    echo "✅ Staging refresh completed successfully"
    # Notify team
    curl -X POST "$SLACK_WEBHOOK" -d '{"text":"🔄 Staging environment refreshed with latest data"}'
else
    echo "❌ Staging refresh failed"
    exit 1
fi
```

## Advanced Workflows

### Snapshot with Pre/Post Processing

```yaml title="Advanced target with processing"
targets:
  advanced_app:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/development
      # Custom processing
      pre_build_script: "./scripts/prepare-snapshot.sh"
      post_load_script: "./scripts/setup-dev-data.sh"
```

**Pre-build script** (`./scripts/prepare-snapshot.sh`):
```bash
#!/bin/bash
# Prepare database before snapshot
echo "Preparing database for snapshot..."

# Create consistent snapshot point
psql "$1" -c "SELECT pg_start_backup('dbsnapper_snapshot');"

# Clean up temporary data
psql "$1" -c "TRUNCATE temp_processing_queue;"
psql "$1" -c "DELETE FROM sessions WHERE last_activity < NOW() - INTERVAL '1 day';"

echo "Database prepared for snapshot"
```

**Post-load script** (`./scripts/setup-dev-data.sh`):
```bash
#!/bin/bash
# Set up development environment after load
echo "Setting up development environment..."

# Create test users
psql "$1" -c "
INSERT INTO users (email, role, created_at) VALUES 
    ('dev@example.com', 'admin', NOW()),
    ('test@example.com', 'user', NOW())
ON CONFLICT (email) DO NOTHING;
"

# Set development configuration
psql "$1" -c "
UPDATE app_settings 
SET value = 'development' 
WHERE key = 'environment';
"

echo "Development environment ready"
```

### Snapshot Versioning Strategy

```bash
# Create versioned snapshots for releases
dbsnapper build my_app --tag "v1.2.0-release"
dbsnapper build my_app --tag "v1.2.0-hotfix"

# Create dated snapshots for regular backups
dbsnapper build my_app --tag "daily-$(date +%Y%m%d)"
dbsnapper build my_app --tag "weekly-$(date +%Y-W%V)"

# Create feature-specific snapshots
dbsnapper build my_app --tag "feature-payment-gateway"
dbsnapper build my_app --tag "before-migration-$(date +%Y%m%d)"
```

### Snapshot Size Optimization

For large databases, optimize snapshot size and speed:

```yaml title="Large database optimization"
targets:
  large_db:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/large_production
      dst_url: postgresql://postgres:password@localhost:5432/large_development
      # Optimization settings
      compression_level: 9
      parallel_workers: 4
      exclude_tables:
        - audit_logs
        - session_data
        - temp_processing
      data_only_tables:
        - large_analytics_table
```

**Build optimized snapshot:**
```bash
# Create optimized snapshot for large database
dbsnapper build large_db \
    --compression-level 9 \
    --parallel-workers 4 \
    --exclude-data "audit_logs,session_data" \
    --tag "optimized-$(date +%Y%m%d)"
```

## Performance Optimization

### Database-Specific Optimizations

#### PostgreSQL Optimization
```yaml
targets:
  postgres_optimized:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/production?application_name=dbsnapper
      dst_url: postgresql://postgres:password@localhost:5432/development
      # PostgreSQL-specific options
      pg_dump_options: "--verbose --compress=9 --jobs=4"
      pg_restore_options: "--verbose --jobs=4 --disable-triggers"
```

#### MySQL Optimization
```yaml
targets:
  mysql_optimized:
    snapshot:
      src_url: mysql://root:password@localhost:3306/production?charset=utf8mb4
      dst_url: mysql://root:password@localhost:3306/development
      # MySQL-specific options
      mysqldump_options: "--single-transaction --routines --triggers --compress"
      mysql_options: "--init-command='SET foreign_key_checks=0;'"
```

### Performance Monitoring

**Monitor snapshot performance:**
```bash
# Build with timing and verbose output
time dbsnapper build my_app --verbose

# Monitor disk usage during snapshot
watch -n 5 'df -h ~/.dbsnapper && du -sh ~/.dbsnapper/*'

# Check database load during snapshot
while dbsnapper build my_app; do
    echo "Snapshot completed at $(date)"
    sleep 300  # Wait 5 minutes between snapshots
done
```

### Large Database Strategies

For databases over 100GB:

```bash
# Use parallel processing
dbsnapper build large_db --parallel-workers 8

# Use subset for development
dbsnapper subset large_db --tag "dev-subset"

# Use compression optimization
dbsnapper build large_db --compression-level 9 --tag "compressed"

# Schedule during low-traffic periods
echo "0 2 * * * dbsnapper build large_db --tag 'nightly-$(date +\%Y\%m\%d)'" | crontab -
```

## Snapshot Management

### Cleanup and Retention

**Automatic cleanup script:**
```bash
#!/bin/bash
# cleanup-snapshots.sh

TARGET="my_app"
KEEP_DAILY=7
KEEP_WEEKLY=4
KEEP_MONTHLY=12

echo "Cleaning up snapshots for $TARGET..."

# Keep last 7 daily snapshots
dbsnapper snapshots $TARGET --older-than "7 days" --tag "daily-*" --delete

# Keep last 4 weekly snapshots
dbsnapper snapshots $TARGET --older-than "4 weeks" --tag "weekly-*" --delete

# Keep last 12 monthly snapshots
dbsnapper snapshots $TARGET --older-than "12 months" --tag "monthly-*" --delete

echo "Cleanup completed"
```

### Snapshot Verification

**Verify snapshot integrity:**
```bash
# Test snapshot by loading to temporary database
dbsnapper load my_app 0 --dst-override "postgresql://postgres:postgres@localhost:5432/test_verify"

# Run basic verification queries
psql postgresql://postgres:postgres@localhost:5432/test_verify -c "
SELECT 
    schemaname,
    tablename,
    n_tup_ins as inserted_rows,
    n_tup_upd as updated_rows,
    n_tup_del as deleted_rows
FROM pg_stat_user_tables 
ORDER BY schemaname, tablename;
"

# Clean up test database
dropdb --if-exists test_verify
```

### Snapshot Comparison

**Compare snapshots to detect changes:**
```bash
# Compare schema between snapshots
dbsnapper compare my_app 0 1 --schema-only

# Compare data counts between snapshots
dbsnapper compare my_app 0 1 --row-counts

# Generate diff report
dbsnapper compare my_app 0 1 --output report.html
```

## Troubleshooting

### Common Build Issues

#### Connection Problems
```bash
# Test database connectivity
dbsnapper targets my_app

# Debug connection with verbose output
dbsnapper build my_app --verbose --debug

# Test connection manually
psql "postgresql://user:pass@host:port/database" -c "SELECT version();"
```

#### Permission Issues
```bash
# Check database permissions
psql "postgresql://user:pass@host:port/database" -c "
SELECT 
    schemaname,
    tablename,
    tableowner,
    has_table_privilege(current_user, schemaname||'.'||tablename, 'SELECT') as can_read
FROM pg_tables 
WHERE has_table_privilege(current_user, schemaname||'.'||tablename, 'SELECT') = false;
"
```

#### Disk Space Issues
```bash
# Check available space
df -h ~/.dbsnapper

# Monitor space during build
watch -n 5 'df -h ~/.dbsnapper && du -sh ~/.dbsnapper/*'

# Clean up old snapshots
find ~/.dbsnapper -name "*.zip" -mtime +30 -delete
```

### Common Load Issues

#### Database Recreation Failures
```bash
# Check destination database permissions
psql "postgresql://admin:pass@host:port/postgres" -c "
SELECT 
    datname,
    datallowconn,
    datconnlimit,
    has_database_privilege(current_user, datname, 'CREATE') as can_create
FROM pg_database 
WHERE datname = 'your_database';
"
```

#### Large Database Timeouts
```yaml
targets:
  large_db:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/large_db
      dst_url: postgresql://postgres:password@localhost:5432/large_dev
      # Increase timeouts for large databases
      connection_timeout: 300
      statement_timeout: 3600
```

#### Memory Issues During Load
```bash
# Load with memory limits
dbsnapper load my_app 0 --memory-limit 2GB

# Use parallel restore for faster loading
dbsnapper load my_app 0 --parallel-workers 4

# Monitor memory usage
watch -n 5 'free -h && ps aux | grep dbsnapper'
```

### Debug Mode

Enable debug mode for detailed troubleshooting:

```bash
# Run with maximum verbosity
DBSNAPPER_DEBUG=true dbsnapper build my_app --verbose

# Save debug output to file
dbsnapper build my_app --debug 2>&1 | tee debug.log

# Check logs for errors
grep -i error debug.log
grep -i warning debug.log
```

## Automation and Scheduling

### Cron Automation

**Daily snapshot schedule:**
```bash
# Add to crontab (crontab -e)
# Daily snapshot at 2 AM
0 2 * * * /usr/local/bin/dbsnapper build production --tag "daily-$(date +\%Y\%m\%d)" >> /var/log/dbsnapper.log 2>&1

# Weekly snapshot on Sundays at 3 AM
0 3 * * 0 /usr/local/bin/dbsnapper build production --tag "weekly-$(date +\%Y-W\%V)" >> /var/log/dbsnapper.log 2>&1

# Monthly snapshot on the 1st at 4 AM
0 4 1 * * /usr/local/bin/dbsnapper build production --tag "monthly-$(date +\%Y\%m)" >> /var/log/dbsnapper.log 2>&1
```

### GitHub Actions Integration

```yaml title=".github/workflows/snapshot.yml"
name: Database Snapshot
on:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM UTC
  workflow_dispatch:

jobs:
  snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install DBSnapper
        uses: dbsnapper/install-dbsnapper-agent-action@v1
        with:
          version: '2.0.3'
      
      - name: Configure DBSnapper
        run: |
          mkdir -p ~/.config/dbsnapper
          echo "${{ secrets.DBSNAPPER_CONFIG }}" > ~/.config/dbsnapper/dbsnapper.yml
      
      - name: Create Snapshot
        run: |
          dbsnapper build production --tag "github-$(date +%Y%m%d)"
        env:
          DBSNAPPER_AUTHTOKEN: ${{ secrets.DBSNAPPER_TOKEN }}
      
      - name: Verify Snapshot
        run: |
          dbsnapper target production | grep -q "Available"
```

### Docker Automation

```dockerfile
# Dockerfile for scheduled snapshots
FROM dbsnapper/dbsnapper:latest

# Copy configuration
COPY dbsnapper.yml /root/.config/dbsnapper/

# Create snapshot script
COPY snapshot.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/snapshot.sh

# Schedule with cron
RUN echo "0 2 * * * /usr/local/bin/snapshot.sh" | crontab -

CMD ["crond", "-f"]
```

```bash title="snapshot.sh"
#!/bin/bash
# Docker snapshot script

# Create snapshot
dbsnapper build production --tag "docker-$(date +%Y%m%d)"

# Upload to cloud if configured
if [ ! -z "$DBSNAPPER_AUTHTOKEN" ]; then
    dbsnapper cloud upload production latest
fi

# Clean up old local snapshots
find ~/.dbsnapper -name "*.zip" -mtime +7 -delete
```

## Best Practices

### Security Best Practices

1. **Use Read-Only Accounts**: Create dedicated read-only database users for snapshots
2. **Sanitize Sensitive Data**: Never snapshot production data without sanitization
3. **Secure Storage**: Use encrypted storage for snapshot files
4. **Access Control**: Limit who can access snapshot files and destination databases

### Operational Best Practices

1. **Regular Testing**: Test snapshot loads regularly to ensure recoverability
2. **Monitor Performance**: Track snapshot size and build times over time
3. **Document Dependencies**: Keep track of which applications depend on specific snapshots
4. **Version Control**: Tag snapshots with meaningful versions and dates

### Development Team Best Practices

1. **Consistent Environments**: Use the same snapshot across the development team
2. **Fresh Data**: Refresh development snapshots regularly (weekly/monthly)
3. **Feature Isolation**: Use separate snapshots for different features when needed
4. **Testing Data**: Create specific snapshots for automated testing

## Next Steps

### Basic Users
1. **[Add Sanitization](../sanitize/introduction.md)** - Remove sensitive data from snapshots
2. **[Set Up Subsetting](../subset/introduction.md)** - Create smaller, focused snapshots
3. **[Configure Cloud Storage](../cloud-storage-engines/introduction.md)** - Share snapshots with your team

### Advanced Users
1. **[Automate with CI/CD](../articles/dbsnapper-github-actions-ecs-simplified.md)** - Integrate into your deployment pipeline
2. **[Team Collaboration](../dbsnapper-cloud/introduction.md)** - Set up DBSnapper Cloud for team sharing
3. **[Performance Tuning](../configuration.md#performance-optimization)** - Optimize for large databases

---

**Need help?** Check our [troubleshooting guide](../troubleshooting.md) or [contact support](https://dbsnapper.com/contact).