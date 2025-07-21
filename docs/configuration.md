---
title: Configuration Guide
description: Complete guide to configuring DBSnapper for your database snapshot workflows - from basic setup to advanced enterprise features.
---

# Configuration Guide

DBSnapper uses a YAML configuration file to define your database targets, storage profiles, and system settings. This guide will take you from basic setup to advanced enterprise configurations.

<!-- prettier-ignore-start -->
!!! tip "Quick Start"

    **New to DBSnapper?** Start with the [Basic Configuration](#basic-configuration) section below.
    
    **Already using DBSnapper?** Jump to [Advanced Configuration](#advanced-configuration) or use the navigation menu to find specific topics.
<!-- prettier-ignore-end -->

## Configuration File Location

The DBSnapper configuration file is located at:

- **Linux/macOS**: `~/.config/dbsnapper/dbsnapper.yml`
- **Windows (WSL2)**: `~/.config/dbsnapper/dbsnapper.yml`

Create this file using:

```bash
dbsnapper config init
```

## Basic Configuration

Start with these essential configurations to get DBSnapper working quickly.

### Minimal Working Example

```yaml title="~/.config/dbsnapper/dbsnapper.yml" linenums="1"
# Basic DBSnapper configuration
secret_key: 1234567890abcdef1234567890abcdef
working_directory: ~/.dbsnapper

# Your first target
targets:
  my_app:
    snapshot:
      src_url: postgresql://user:password@localhost:5432/production_db
      dst_url: postgresql://user:password@localhost:5432/development_db
```

<!-- prettier-ignore-start -->
!!! success "What This Does"

    This basic configuration:
    
    - Sets up encryption with a secret key
    - Defines where snapshots are stored locally
    - Creates one target called `my_app`
    - Connects to your production database as source
    - Loads snapshots into your development database
<!-- prettier-ignore-end -->

### Essential Settings Explained

#### Secret Key
```yaml
secret_key: 1234567890abcdef1234567890abcdef
```
- **Purpose**: Encrypts your snapshots for security
- **Format**: 32-character hexadecimal string (16 bytes)
- **Auto-generated**: Created by `dbsnapper config init`
- **Environment**: Can use `DBSNAPPER_SECRET_KEY` instead

#### Working Directory
```yaml
working_directory: ~/.dbsnapper
```
- **Purpose**: Where snapshots are stored locally
- **Default**: `~/.dbsnapper`
- **Requirements**: Must have write permissions
- **Space**: Ensure enough disk space (2x your largest database)

#### Basic Target Definition
```yaml
targets:
  my_app:  # Target name (choose any identifier)
    snapshot:
      src_url: postgresql://user:pass@host:port/database
      dst_url: postgresql://user:pass@host:port/database
```

### Common Database Connection Examples

#### PostgreSQL
```yaml
targets:
  postgres_app:
    snapshot:
      src_url: postgresql://postgres:mypassword@localhost:5432/production?sslmode=require
      dst_url: postgresql://postgres:mypassword@localhost:5432/development?sslmode=disable
```

#### MySQL
```yaml
targets:
  mysql_app:
    snapshot:
      src_url: mysql://root:mypassword@localhost:3306/production?tls=true
      dst_url: mysql://root:mypassword@localhost:3306/development?tls=false
```

#### Using Environment Variables
```yaml
targets:
  secure_app:
    snapshot:
      src_url: postgresql://{{`DB_USER` | env}}:{{`DB_PASS` | env}}@{{`DB_HOST` | env}}/{{`DB_NAME` | env}}
      dst_url: postgresql://postgres:postgres@localhost:5432/development
```

<!-- prettier-ignore-start -->
!!! warning "Destination Database Safety"

    **The destination database will be completely dropped and recreated** when loading snapshots.
    
    **Never use a production database as a destination!**
    
    Always verify your `dst_url` points to a development/testing database.
<!-- prettier-ignore-end -->

### Validation and Testing

After creating your configuration, validate it:

```bash
# Check configuration syntax and database connections
dbsnapper config check

# List configured targets
dbsnapper targets

# Test connection to a specific target
dbsnapper targets my_app
```

Expected `config check` output:
```
✅ Config file found and loaded
✅ PostgreSQL tools available
✅ Target 'my_app' source: Connected
✅ Target 'my_app' destination: Connected
✅ Configuration OK
```

## Intermediate Configuration

Add these features as your needs grow.

### Docker Database Support

If you use Docker for databases, add Docker engine configuration:

```yaml
# Enable Docker-based database engines
docker:
  images:
    postgres: postgres:16-alpine
    mysql: mysql:8.0

targets:
  docker_app:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5433/development
    # Use Docker engine instead of local tools
    engine: pgdocker
```

### Basic Data Sanitization

Remove sensitive data during snapshots:

```yaml
targets:
  ecommerce_app:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/development
    # Add sanitization
    sanitize:
      dst_url: postgresql://postgres:password@localhost:5432/sanitized_dev
      queries:
        - "UPDATE users SET email = 'user' || id || '@example.com'"
        - "UPDATE users SET ssn = 'XXX-XX-' || RIGHT(ssn, 4)"
        - "UPDATE orders SET credit_card = 'XXXX-XXXX-XXXX-' || RIGHT(credit_card, 4)"
```

### Basic Data Subsetting

Create smaller snapshots with related data:

```yaml
targets:
  large_app:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/development
    # Add subsetting for large databases
    subset:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/subset_dev
      subset_tables:
        - table: public.users
          where: "created_at > NOW() - INTERVAL '30 days'"
        - table: public.orders
          percent: 10
      copy_tables:
        - public.products
        - public.categories
```

### Environment Variable Configuration

For CI/CD or containerized deployments, use environment variables:

```bash
# Set via environment instead of config file
export DBSNAPPER_SECRET_KEY="1234567890abcdef1234567890abcdef"
export DBSNAPPER_WORKING_DIRECTORY="/tmp/dbsnapper"
export DBSNAPPER_AUTHTOKEN="your-cloud-token"

# Target-specific environment variables
export DBSNAPPER_TARGETS__MY_APP__SNAPSHOT__SRC_URL="postgresql://..."
export DBSNAPPER_TARGETS__MY_APP__SNAPSHOT__DST_URL="postgresql://..."
```

Environment variable naming convention:
- Prefix with `DBSNAPPER_`
- Replace periods with double underscores `__`
- Use uppercase

Examples:
- `docker.images.postgres` → `DBSNAPPER_DOCKER__IMAGES__POSTGRES`
- `targets.my_app.snapshot.src_url` → `DBSNAPPER_TARGETS__MY_APP__SNAPSHOT__SRC_URL`

## Advanced Configuration

Enterprise features for production environments.

### DBSnapper Cloud Integration

Connect to DBSnapper Cloud for team sharing and managed storage:

```yaml
# Cloud authentication
authtoken: your-cloud-auth-token-here

# Default destination for shared snapshots
defaults:
  shared_target_dst_db_url: postgresql://postgres:password@localhost:5432/shared_snapshots

# Cloud storage profiles
storage_profiles:
  production_s3:
    provider: s3
    awscli_profile: dbsnapper-prod
    bucket: company-dbsnapper-snapshots
    prefix: production/
    region: us-east-1
  
  development_s3:
    provider: s3
    access_key: AKIA...
    secret_key: secret...
    bucket: company-dbsnapper-dev
    prefix: development/
    region: us-west-2

# Sharing targets
targets:
  shared_production:
    share:
      dst_url: postgresql://postgres:password@localhost:5432/shared_prod
      storage_profile_name: production_s3
```

### Advanced Sanitization Patterns

#### GDPR Compliance
```yaml
targets:
  gdpr_compliant:
    sanitize:
      dst_url: postgresql://postgres:password@localhost:5432/gdpr_safe
      queries:
        # Personal identifiers
        - "UPDATE users SET email = 'deleted_' || id || '@example.com'"
        - "UPDATE users SET first_name = 'User', last_name = id::text"
        - "UPDATE users SET phone = NULL, address = NULL"
        
        # Right to be forgotten
        - "DELETE FROM user_activity WHERE user_id IN (SELECT id FROM users WHERE gdpr_delete_requested = true)"
        - "UPDATE users SET email = 'deleted@example.com', status = 'deleted' WHERE gdpr_delete_requested = true"
        
        # Data minimization
        - "UPDATE users SET birth_date = NULL WHERE created_at < NOW() - INTERVAL '2 years'"
```

#### HIPAA Compliance
```yaml
targets:
  hipaa_safe:
    sanitize:
      dst_url: postgresql://postgres:password@localhost:5432/hipaa_compliant
      queries:
        # PHI removal
        - "UPDATE patients SET ssn = 'XXX-XX-' || RIGHT(ssn, 4)"
        - "UPDATE patients SET first_name = 'Patient', last_name = id::text"
        - "UPDATE patients SET email = 'patient' || id || '@hospital.local'"
        - "UPDATE patients SET phone = '555-0' || LPAD((id % 10000)::text, 3, '0')"
        
        # Medical record sanitization
        - "UPDATE medical_records SET notes = 'Sanitized for development use'"
        - "UPDATE prescriptions SET notes = NULL"
        
        # Audit trail preservation
        - "INSERT INTO sanitization_log (table_name, record_count, sanitized_at) VALUES ('patients', (SELECT COUNT(*) FROM patients), NOW())"
```

#### Financial Data Sanitization
```yaml
targets:
  financial_safe:
    sanitize:
      dst_url: postgresql://postgres:password@localhost:5432/financial_dev
      queries:
        # PCI DSS compliance
        - "UPDATE credit_cards SET number = '4111111111111' || RIGHT(number, 4)"
        - "UPDATE credit_cards SET cvv = '123', expiry_date = '12/25'"
        
        # Account number masking
        - "UPDATE bank_accounts SET account_number = 'XXXX-XXXX-' || RIGHT(account_number, 4)"
        - "UPDATE bank_accounts SET routing_number = '123456789'"
        
        # Transaction amount obfuscation (preserving patterns)
        - "UPDATE transactions SET amount = ROUND((amount * 0.1 + (RANDOM() * amount * 0.2))::numeric, 2)"
        
        # SSN handling
        - "UPDATE customers SET ssn = 'XXX-XX-' || RIGHT(ssn, 4)"
```

### Complex Subsetting Strategies

#### Time-Based Subsetting
```yaml
targets:
  recent_data:
    subset:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/recent_subset
      subset_tables:
        # Recent orders with cascading relationships
        - table: public.orders
          where: "created_at > NOW() - INTERVAL '90 days'"
        
        # Active users only
        - table: public.users
          where: "last_login > NOW() - INTERVAL '180 days' AND status = 'active'"
        
        # Sample of products (for variety)
        - table: public.products
          percent: 25
      
      # Always include reference data
      copy_tables:
        - public.countries
        - public.currencies
        - public.product_categories
        - public.payment_methods
      
      # Skip large audit tables
      excluded_tables:
        - public.audit_logs
        - public.session_tracking
        - public.email_logs
```

#### Geographic Subsetting
```yaml
targets:
  regional_subset:
    subset:
      src_url: postgresql://postgres:password@localhost:5432/production
      dst_url: postgresql://postgres:password@localhost:5432/us_west_subset
      subset_tables:
        # Focus on specific geographic regions
        - table: public.stores
          where: "region IN ('US-WEST', 'US-CENTRAL')"
        
        # Customers in those regions
        - table: public.customers
          where: "country = 'US' AND state IN ('CA', 'OR', 'WA', 'NV', 'AZ', 'CO')"
        
        # Orders from those stores/customers (handled by relationships)
        
      # Define relationships if not in schema
      added_relationships:
        - fk_table: public.orders
          fk_columns: store_id
          ref_table: public.stores
          ref_columns: id
        - fk_table: public.orders
          fk_columns: customer_id
          ref_table: public.customers
          ref_columns: id
```

### Multi-Environment Configuration

#### Development Team Setup
```yaml
# Shared team configuration
authtoken: team-shared-token

defaults:
  shared_target_dst_db_url: postgresql://postgres:password@localhost:5432/team_shared

targets:
  # Production snapshots (read-only for developers)
  production_readonly:
    share:
      dst_url: postgresql://postgres:password@localhost:5432/prod_readonly
      storage_profile_name: production_s3_readonly
  
  # Staging environment
  staging:
    snapshot:
      src_url: postgresql://staging_user:password@staging.company.com:5432/app
      dst_url: postgresql://postgres:password@localhost:5432/staging_local
    sanitize:
      dst_url: postgresql://postgres:password@localhost:5432/staging_sanitized
      queries:
        - "UPDATE users SET email = 'dev' || id || '@company-staging.com'"
  
  # Feature development
  feature_branch:
    snapshot:
      src_url: postgresql://postgres:password@localhost:5432/staging_sanitized
      dst_url: postgresql://postgres:password@localhost:5432/feature_dev
    subset:
      src_url: postgresql://postgres:password@localhost:5432/staging_sanitized
      dst_url: postgresql://postgres:password@localhost:5432/feature_subset
      subset_tables:
        - table: public.users
          percent: 5
        - table: public.orders
          where: "created_at > NOW() - INTERVAL '30 days'"

storage_profiles:
  production_s3_readonly:
    provider: s3
    awscli_profile: company-prod-readonly
    bucket: company-prod-snapshots
    prefix: sanitized/
    region: us-east-1
```

### Configuration Overrides

For specific deployment scenarios:

```yaml
# Global overrides (applied to all operations)
override:
  # Custom sanitization query (base64 encoded)
  san_query: VVBEQVRFIHVzZXJzIFNFVCBlbWFpbCA9ICdkZXYnIHx8IGlkIHx8ICdAZXhhbXBsZS5jb20n
  
  # Override destination database for all operations
  dst_db_url: postgresql://postgres:password@localhost:15432/override_destination

# Environment-specific defaults
defaults:
  shared_target_dst_db_url: postgresql://postgres:password@localhost:5432/shared_default
```

## Configuration Templates

### E-commerce Application
```yaml title="ecommerce-template.yml"
secret_key: your-secret-key-here
working_directory: ~/.dbsnapper

docker:
  images:
    postgres: postgres:16-alpine
    mysql: mysql:8.0

targets:
  ecommerce_prod:
    snapshot:
      src_url: postgresql://{{`PROD_DB_USER` | env}}:{{`PROD_DB_PASS` | env}}@{{`PROD_DB_HOST` | env}}/ecommerce
      dst_url: postgresql://postgres:postgres@localhost:5432/ecommerce_dev
    
    sanitize:
      dst_url: postgresql://postgres:postgres@localhost:5432/ecommerce_sanitized
      queries:
        # Customer data
        - "UPDATE customers SET email = 'customer' || id || '@example.com'"
        - "UPDATE customers SET first_name = 'Customer', last_name = id::text"
        - "UPDATE customers SET phone = '555-0' || LPAD((id % 1000)::text, 3, '0')"
        
        # Payment data
        - "UPDATE payment_methods SET card_number = '4111111111111' || RIGHT(card_number, 4)"
        - "UPDATE payment_methods SET cvv = '123'"
        
        # Order amounts (preserve patterns)
        - "UPDATE orders SET total_amount = ROUND((total_amount * 0.8 + RANDOM() * total_amount * 0.4)::numeric, 2)"
    
    subset:
      src_url: postgresql://postgres:postgres@localhost:5432/ecommerce_sanitized
      dst_url: postgresql://postgres:postgres@localhost:5432/ecommerce_subset
      subset_tables:
        - table: public.customers
          where: "created_at > NOW() - INTERVAL '6 months'"
        - table: public.products
          percent: 30
      copy_tables:
        - public.categories
        - public.brands
        - public.countries
```

### SaaS Multi-Tenant Application
```yaml title="saas-template.yml"
secret_key: your-secret-key-here
authtoken: your-cloud-token-here

targets:
  saas_tenant_demo:
    snapshot:
      src_url: postgresql://{{`DB_USER` | env}}:{{`DB_PASS` | env}}@{{`DB_HOST` | env}}/saas_prod
      dst_url: postgresql://postgres:postgres@localhost:5432/saas_demo
    
    sanitize:
      dst_url: postgresql://postgres:postgres@localhost:5432/saas_demo_sanitized
      queries:
        # Tenant isolation preserved
        - "UPDATE users SET email = tenant_id || '_user' || (id % 1000) || '@demo.com'"
        - "UPDATE users SET first_name = 'Demo', last_name = 'User' || (id % 1000)"
        
        # Organization data
        - "UPDATE organizations SET name = 'Demo Org ' || id"
        - "UPDATE organizations SET domain = 'demo-org-' || id || '.example.com'"
        
        # Preserve tenant relationships but sanitize content
        - "UPDATE tenant_data SET sensitive_field = 'demo_value_' || (id % 100)"
    
    subset:
      src_url: postgresql://postgres:postgres@localhost:5432/saas_demo_sanitized
      dst_url: postgresql://postgres:postgres@localhost:5432/saas_demo_subset
      subset_tables:
        # Keep 2-3 demo tenants with full data
        - table: public.organizations
          where: "id IN (1, 2, 3)"
      
      # Exclude tenant-specific large tables
      excluded_tables:
        - public.audit_logs
        - public.usage_metrics
        - public.backup_data

storage_profiles:
  demo_storage:
    provider: s3
    bucket: company-demo-snapshots
    prefix: sanitized/
    awscli_profile: demo-profile
```

### Development Team Template
```yaml title="team-template.yml"
# Team shared configuration
authtoken: team-shared-token-here
secret_key: team-shared-secret-key

defaults:
  shared_target_dst_db_url: postgresql://postgres:postgres@localhost:5432/team_shared

targets:
  # Production snapshots (shared, sanitized)
  prod_shared:
    share:
      dst_url: postgresql://postgres:postgres@localhost:5432/prod_shared
      storage_profile_name: team_s3
  
  # Personal development database
  personal_dev:
    snapshot:
      src_url: postgresql://postgres:postgres@localhost:5432/prod_shared
      dst_url: postgresql://postgres:postgres@localhost:5432/{{`USER` | env}}_dev
    
    subset:
      src_url: postgresql://postgres:postgres@localhost:5432/prod_shared
      dst_url: postgresql://postgres:postgres@localhost:5432/{{`USER` | env}}_subset
      subset_tables:
        - table: public.users
          percent: 1
        - table: public.orders
          where: "created_at > NOW() - INTERVAL '7 days'"

storage_profiles:
  team_s3:
    provider: s3
    awscli_profile: team-profile
    bucket: team-snapshots
    prefix: shared/sanitized/
```

## Troubleshooting Configuration

### Common Configuration Issues

#### YAML Syntax Errors
```bash
# Validate YAML syntax
python -c "import yaml; print('Valid YAML' if yaml.safe_load(open('~/.config/dbsnapper/dbsnapper.yml')) else 'Invalid YAML')"

# Check with DBSnapper
dbsnapper config check
```

Common YAML mistakes:
- **Indentation**: Use spaces, not tabs
- **Quoting**: Quote values with special characters
- **Lists**: Use `- item` format with proper indentation

#### Connection String Issues
```bash
# Test database connectivity separately
psql "postgresql://user:pass@host:port/database" -c "SELECT version();"
mysql -h host -P port -u user -p database -e "SELECT VERSION();"

# Check DBSnapper connection
dbsnapper targets target_name
```

#### Environment Variable Problems
```bash
# Check environment variables
env | grep DBSNAPPER

# Test template expansion
dbsnapper config check --debug
```

#### Permission Issues
```bash
# Check file permissions
ls -la ~/.config/dbsnapper/

# Ensure directory is writable
touch ~/.config/dbsnapper/test && rm ~/.config/dbsnapper/test

# Check working directory permissions
mkdir -p ~/.dbsnapper && touch ~/.dbsnapper/test && rm ~/.dbsnapper/test
```

### Configuration Validation Commands

```bash
# Comprehensive configuration check
dbsnapper config check

# Verify specific target
dbsnapper targets my_target

# Test connection strings
dbsnapper config check --target my_target --verbose

# Validate storage profiles
dbsnapper config check --storage-profiles

# Check environment variable expansion
dbsnapper config dump --resolve-env
```

### Performance Optimization

#### Large Database Configuration
```yaml
# Optimize for large databases (>100GB)
targets:
  large_db:
    snapshot:
      src_url: postgresql://user:pass@host/large_db?sslmode=require
      dst_url: postgresql://user:pass@localhost/large_db_local?sslmode=disable
      # Increase timeout for large databases
      timeout: 3600
      # Use compression
      compression: true
      # Parallel processing
      parallel_workers: 4

working_directory: /data/dbsnapper  # Use fast storage
```

#### Memory Usage Optimization
```yaml
# For memory-constrained environments
working_directory: /tmp/dbsnapper  # Use temporary storage
docker:
  memory_limit: 2g  # Limit Docker memory usage
  images:
    postgres: postgres:16-alpine  # Use smaller images
```

## Next Steps

### Basic Users
1. **[Create Your First Snapshot](quick-start.md)** - Test your configuration
2. **[Add Sanitization](sanitize/introduction.md)** - Remove sensitive data
3. **[Set Up Team Sharing](dbsnapper-cloud/introduction.md)** - Collaborate securely

### Advanced Users
1. **[Subset Configuration](subset/introduction.md)** - Optimize snapshot size
2. **[Storage Profiles](cloud-storage-engines/introduction.md)** - Configure cloud storage
3. **[CI/CD Integration](articles/dbsnapper-github-actions-ecs-simplified.md)** - Automate workflows

### Enterprise Teams
1. **[DBSnapper Cloud Setup](dbsnapper-cloud/introduction.md)** - Team management
2. **[SSO Configuration](dbsnapper-cloud/sso/sso-okta-oidc.md)** - Enterprise authentication
3. **[Compliance Documentation](sanitize/configuration.md)** - Regulatory requirements

---

**Need help?** Check our [troubleshooting guide](troubleshooting.md) or [contact support](https://dbsnapper.com/contact).