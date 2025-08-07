# Subset Command

The `subset` command creates a relationally-complete subset of a database, maintaining all critical relationships and constraints while reducing the overall data size. This powerful feature enables you to create smaller, more manageable versions of large databases for development, testing, and analysis purposes.

## Overview

The subset command provides intelligent database subsetting:
- **Relational integrity**: Maintains all foreign key relationships and constraints
- **Configurable criteria**: Define subset rules using WHERE clauses or percentage sampling
- **Automatic relationship resolution**: Intelligently includes related data to maintain consistency
- **Performance optimization**: Efficiently processes large databases with complex relationships
- **Flexible table handling**: Support for copy tables, excluded tables, and manual relationships

## Syntax

```bash
dbsnapper subset <target_name> [flags]
```

## Arguments

- `<target_name>`: Name of the target with subset configuration (defined in configuration file)

## Options

```bash
  -h, --help   help for subset

Global Flags:
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

## Subset Configuration Requirements

The subset command requires a target with a `subset` configuration section:

```yaml
targets:
  sample-target:
    name: sample-target
    subset:
      src_url: postgres://user:pass@localhost:5432/large_database
      dst_url: postgres://user:pass@localhost:5432/subset_database
      subset_tables:
        - table: public.orders
          where: "created_at >= '2024-01-01'"
        - table: public.customers
          percent: 10
      copy_tables:
        - public.lookup_tables
      excluded_tables:
        - public.audit_logs
```

## How It Works

### Subsetting Process Overview

The subset command follows a systematic approach to create relationally-complete subsets:

#### 1. Database Analysis Phase
- **Schema analysis**: Examines database structure and relationships
- **Constraint discovery**: Identifies foreign key constraints and relationships
- **Dependency mapping**: Creates a relationship graph for processing order
- **Validation**: Ensures subset configuration is valid and feasible

#### 2. Subset Table Processing
- **Primary filtering**: Applies WHERE clauses or percentage sampling to subset tables
- **Data extraction**: Extracts qualifying records from each subset table
- **Key collection**: Collects primary keys for relationship resolution
- **Validation**: Ensures extracted data meets quality requirements

#### 3. Upstream Table Resolution
- **Dependency analysis**: Identifies tables that reference subset table data
- **Relationship following**: Follows foreign key relationships upward
- **Recursive processing**: Continues until all upstream dependencies resolved
- **Circular reference detection**: Identifies and handles circular relationships

#### 4. Copy Table Processing
- **Full table copy**: Copies designated tables entirely to destination
- **Structure preservation**: Maintains all table structures and constraints
- **Data integrity**: Ensures copy tables integrate with subset data
- **Performance optimization**: Uses efficient bulk copy operations

#### 5. Downstream Table Resolution
- **Reference resolution**: Identifies tables referenced by subset data
- **Selective copying**: Copies only referenced records from downstream tables
- **Constraint validation**: Ensures all foreign key constraints are satisfied
- **Final validation**: Verifies subset completeness and integrity

### Subset Table Configuration

#### WHERE Clause Subsetting
```yaml
subset_tables:
  - table: public.orders
    where: "created_at >= '2024-01-01' AND status = 'completed'"
  - table: public.products
    where: "category_id IN (1, 2, 3)"
```

**Features:**
- **Complex conditions**: Support for complex SQL WHERE conditions
- **Multiple criteria**: Combine multiple filtering conditions
- **Date filtering**: Common pattern for temporal data subsets
- **Category filtering**: Filter by categories, types, or classifications

#### Percentage Sampling
```yaml
subset_tables:
  - table: public.customers
    percent: 15
  - table: public.transactions
    percent: 5
```

**Features:**
- **Random sampling**: Uses database random sampling for representative data
- **Percentage precision**: Supports decimal percentages (e.g., 2.5)
- **Reproducible results**: Consistent results across multiple runs
- **Performance optimized**: Efficient sampling for large tables

### Table Management Options

#### Copy Tables (Full Copy)
```yaml
copy_tables:
  - public.lookup_countries
  - public.lookup_currencies
  - public.system_configurations
```

**Use cases:**
- **Reference data**: Lookup tables required for application functionality
- **System data**: Configuration and system tables
- **Small critical tables**: Tables that must be complete for functionality
- **Static data**: Tables that don't require subsetting

#### Excluded Tables
```yaml
excluded_tables:
  - public.audit_logs
  - public.system_logs
  - public.temporary_data
```

**Use cases:**
- **Sensitive data**: Tables containing sensitive information
- **Large unnecessary data**: Logs and audit trails not needed for testing
- **Temporary data**: Transient data not required in subset
- **Performance optimization**: Exclude large tables that aren't needed

### Relationship Management

#### Automatic Relationship Discovery
DBSnapper automatically discovers relationships through:
- **Foreign key constraints**: Database-defined foreign key relationships
- **Primary key references**: Identifies primary/foreign key relationships
- **Multi-column relationships**: Supports composite key relationships
- **Cross-schema relationships**: Handles relationships across database schemas

#### Manual Relationship Definition
```yaml
added_relationships:
  - fk_table: public.orders
    fk_columns: customer_id
    ref_table: public.customers
    ref_columns: id
  - fk_table: public.order_items
    fk_columns: product_id
    ref_table: public.products
    ref_columns: product_id
```

**Required when:**
- **Missing constraints**: Database lacks formal foreign key constraints
- **Legacy databases**: Older databases without proper constraint definitions
- **Performance reasons**: Constraints removed for performance but relationships exist
- **Complex relationships**: Non-standard relationship patterns

#### Circular Reference Resolution
```yaml
excluded_relationships:
  - fk_table: public.employees
    ref_table: public.departments
  - fk_table: public.departments
    ref_table: public.employees
```

**Handles:**
- **Self-referencing tables**: Tables with references to themselves
- **Mutual references**: Tables that reference each other
- **Complex cycles**: Multi-table circular reference chains
- **Data integrity**: Maintains data integrity when breaking cycles

## Configuration Examples

### E-commerce Database Subset
```yaml
targets:
  ecommerce-subset:
    name: ecommerce-subset
    subset:
      src_url: postgres://user:pass@prod-server:5432/ecommerce_prod
      dst_url: postgres://user:pass@localhost:5432/ecommerce_dev
      
      # Get recent orders and related customers
      subset_tables:
        - table: public.orders
          where: "created_at >= NOW() - INTERVAL '6 months'"
        - table: public.customers
          where: "last_purchase >= NOW() - INTERVAL '1 year'"
      
      # Copy essential reference data
      copy_tables:
        - public.products
        - public.categories
        - public.shipping_methods
        - public.payment_methods
      
      # Exclude unnecessary data
      excluded_tables:
        - public.audit_logs
        - public.email_logs
        - public.search_analytics
      
      # Define missing relationships
      added_relationships:
        - fk_table: public.order_items
          fk_columns: variant_id
          ref_table: public.product_variants
          ref_columns: id
```

### SaaS Application Subset
```yaml
targets:
  saas-tenant-subset:
    name: saas-tenant-subset
    subset:
      src_url: mysql://user:pass@prod-db:3306/saas_production
      dst_url: mysql://user:pass@localhost:3306/saas_development
      
      # Sample specific tenants
      subset_tables:
        - table: tenants.organizations
          where: "tier = 'enterprise' AND created_at >= '2024-01-01'"
        - table: tenants.users
          percent: 5
      
      # Copy configuration data
      copy_tables:
        - config.features
        - config.pricing_plans
        - config.regions
      
      # Handle tenant isolation
      excluded_relationships:
        - fk_table: tenants.billing_accounts
          ref_table: tenants.organizations
```

### Financial Data Subset
```yaml
targets:
  financial-subset:
    name: financial-subset
    subset:
      src_url: postgres://user:pass@warehouse:5432/financial_data
      dst_url: postgres://user:pass@localhost:5432/financial_test
      
      # Recent transaction data
      subset_tables:
        - table: transactions.payments
          where: "transaction_date >= '2024-01-01' AND amount > 100"
        - table: customers.accounts
          where: "account_type IN ('checking', 'savings')"
      
      # Essential reference data
      copy_tables:
        - reference.currencies
        - reference.exchange_rates
        - reference.bank_codes
      
      # Exclude sensitive/large data
      excluded_tables:
        - audit.transaction_logs
        - compliance.pii_data
        - analytics.large_datasets
      
      # Handle complex financial relationships
      added_relationships:
        - fk_table: transactions.transfers
          fk_columns: from_account_id
          ref_table: customers.accounts
          ref_columns: account_id
        - fk_table: transactions.transfers
          fk_columns: to_account_id
          ref_table: customers.accounts
          ref_columns: account_id
```

## Example Usage

### Basic Database Subset
```bash
# Create subset using configured target
dbsnapper subset ecommerce-dev-data
```

**Output:**
```
🔍 Analyzing database structure...
✅ Found 24 tables with 15 relationships

📊 Processing subset tables:
  → public.orders: Applied WHERE filter, found 5,432 records
  → public.customers: 10% sample, found 1,203 records

🔗 Resolving upstream relationships:
  → public.order_items: Found 12,847 related records
  → public.addresses: Found 2,341 related records
  → public.customer_preferences: Found 1,203 related records

📥 Copying full tables:
  → public.products: 15,678 records
  → public.categories: 245 records
  → public.shipping_methods: 12 records

🔗 Resolving downstream relationships:
  → public.product_variants: Found 8,934 related records
  → public.inventory: Found 15,678 related records

✅ Subset created successfully
📈 Final database: 56,571 records across 18 tables
🎯 Reduction: 89.3% size reduction from original
```

### Large Database Subsetting
```bash
# Subset large production database with progress tracking
dbsnapper subset large-production-data
```

**Output:**
```
🔍 Analyzing large database (2.1TB, 847 tables)...
⚠️  Large database detected, using optimized processing

📊 Processing subset tables (4/4):
  [████████████████████████████████████████] 100%
  → orders: 125,000 records (WHERE: last_6_months)
  → customers: 45,000 records (15% sample)
  → products: 8,500 records (WHERE: active_products)
  → categories: 234 records (25% sample)

🔗 Resolving relationships in batches:
  [████████████████████████████████████████] 100%
  → Processing 156 upstream tables
  → Processing 67 downstream tables
  → Handling 12 copy tables

⏱️  Processing time: 2h 15m
✅ Subset created: 1.2M records (0.05% of original)
💾 Size: 8.7GB (from 2.1TB - 99.6% reduction)
```

### Multi-Schema Database
```bash
# Subset complex multi-schema database
dbsnapper subset multi-schema-app
```

**Output:**
```
🔍 Analyzing multi-schema database...
📂 Found schemas: public, customer_data, products, analytics, audit

📊 Processing subset tables across schemas:
  → customer_data.profiles: 15,000 records (10% sample)
  → products.catalog: WHERE category filter, 2,500 records
  → public.orders: 45,000 records (recent orders)

🔗 Cross-schema relationship resolution:
  → public.orders → customer_data.profiles ✅
  → public.order_items → products.catalog ✅
  → customer_data.addresses → public.orders ✅

📥 Schema-aware copy tables:
  → products.categories: 156 records
  → public.lookup_data: 1,234 records
  → customer_data.preferences: 890 records

✅ Multi-schema subset complete
🎯 Maintained referential integrity across all schemas
```

## Use Cases

### Development Environment Setup
```bash
# Create development database from production subset
dbsnapper subset production-to-dev

# Load into development environment
dbsnapper load development-db
```

**Benefits:**
- **Realistic data**: Work with production-like data structures
- **Performance testing**: Test with representative data volumes  
- **Feature development**: Develop against realistic data relationships
- **Reduced storage**: Significantly smaller than full production copy

### QA/Testing Environment
```bash
# Create consistent test dataset
dbsnapper subset qa-test-data

# Use in automated testing
dbsnapper load automated-test-db
```

**Benefits:**
- **Consistent testing**: Same subset for all test runs
- **Complete relationships**: All data relationships intact
- **Performance**: Faster test execution with smaller datasets
- **Coverage**: Test edge cases with real data patterns

### Demo and Training
```bash
# Create demo database with realistic but manageable size
dbsnapper subset customer-demo-data

# Load for demo environment
dbsnapper load demo-environment
```

**Benefits:**
- **Realistic demos**: Show actual application behavior
- **Manageable size**: Quick to load and reset
- **Data privacy**: Subset can exclude sensitive information
- **Training**: Provide consistent training datasets

### Data Analysis and Reporting
```bash
# Create analytical subset focused on specific metrics
dbsnapper subset analytics-sample

# Load into analytical database
dbsnapper load analytics-warehouse
```

**Benefits:**
- **Focused analysis**: Subset relevant to specific questions
- **Performance**: Faster queries on smaller datasets
- **Cost efficiency**: Reduced cloud computing costs
- **Data freshness**: Recent data for timely analysis

### Database Migration Testing
```bash
# Create subset for migration testing
dbsnapper subset migration-test-data

# Test migration process
dbsnapper load migration-target-db
```

**Benefits:**
- **Risk reduction**: Test migrations on representative subset
- **Faster iteration**: Quick migration testing cycles
- **Issue identification**: Find problems before full migration
- **Validation**: Verify migration results with known data

## Performance Considerations

### Large Database Optimization

#### Batch Processing
```yaml
# Configuration for large databases
subset:
  # Process in smaller batches for memory efficiency
  batch_size: 10000
  
  # Use indexed columns in WHERE clauses
  subset_tables:
    - table: large_table
      where: "indexed_date_column >= '2024-01-01'"  # Uses index
```

#### Memory Management
- **Streaming processing**: Processes data in streams to manage memory
- **Batch commits**: Commits data in batches for performance
- **Connection pooling**: Efficient database connection management
- **Parallel processing**: Utilizes multiple connections for large operations

#### Index Utilization
```yaml
subset_tables:
  # Good: Uses indexed columns
  - table: transactions
    where: "created_at >= '2024-01-01'"  # Assumes index on created_at
  
  # Avoid: Full table scans
  - table: customers
    where: "UPPER(email) LIKE '%@COMPANY.COM'"  # Requires full scan
```

### Network and I/O Optimization

#### Source Database Impact
- **Read-only operations**: Subset operations don't modify source database
- **Low lock impact**: Uses snapshot isolation where possible
- **Off-peak scheduling**: Recommend running during low-traffic periods
- **Connection limits**: Respects database connection limits

#### Network Efficiency
```bash
# Use local network connections when possible
dbsnapper subset local-network-target

# Consider compression for remote connections
# (Handled automatically by database drivers)
```

#### Storage Optimization
- **Destination preparation**: Ensures destination database is properly sized
- **Transaction log management**: Manages transaction logs during large operations
- **Space estimation**: Provides estimates before starting operations
- **Cleanup procedures**: Includes cleanup recommendations after completion

## Database Engine Support

### PostgreSQL Support
**Features:**
- **Schema awareness**: Full support for PostgreSQL schemas
- **Advanced data types**: Handles JSON, arrays, custom types
- **Constraint support**: Foreign keys, check constraints, unique constraints
- **Performance**: Utilizes PostgreSQL-specific optimizations

**Configuration:**
```yaml
subset:
  src_url: postgres://user:pass@host:5432/source_db
  dst_url: postgres://user:pass@host:5432/subset_db
  
  subset_tables:
    - table: public.orders  # Schema.table format
      where: "data->>'status' = 'completed'"  # JSON support
```

### MySQL Support
**Features:**
- **Multi-database support**: Handles multiple MySQL databases
- **Engine compatibility**: Works with InnoDB and other storage engines
- **Character set handling**: Proper UTF-8 and collation support
- **Performance**: MySQL-optimized query generation

**Configuration:**
```yaml
subset:
  src_url: mysql://user:pass@host:3306/source_db
  dst_url: mysql://user:pass@host:3306/subset_db
  
  subset_tables:
    - table: database.orders  # Database.table format
      where: "DATE(created_at) >= '2024-01-01'"  # MySQL date functions
```

### Cross-Platform Considerations
- **SQL dialect differences**: Handles database-specific SQL variations
- **Data type mapping**: Proper handling of database-specific data types
- **Function compatibility**: Uses compatible functions across engines
- **Performance tuning**: Database-specific performance optimizations

## Error Handling and Recovery

### Common Error Scenarios

#### Circular Reference Detection
```
Error: Circular reference detected between tables:
  orders → customers → customer_orders → orders

Resolution: Add excluded_relationships to break the cycle
```

**Solution:**
```yaml
excluded_relationships:
  - fk_table: customer_orders
    ref_table: orders
```

#### Missing Relationships
```
Error: Referenced records not found for foreign key constraint
  Table: order_items, Column: product_id
  Missing references in: products

Resolution: Check subset_tables configuration
```

**Solution:**
```yaml
# Ensure products are included via subset_tables or copy_tables
copy_tables:
  - products
```

#### Configuration Validation
```
Error: Invalid subset configuration
  - subset_tables: must specify either 'where' or 'percent'
  - src_url: database connection failed
  - dst_url: destination database not accessible
```

**Solutions:**
```yaml
subset_tables:
  # Fix: Add either where or percent
  - table: orders
    where: "status = 'completed'"  # Add missing criteria
```

### Recovery Procedures

#### Partial Failure Recovery
```bash
# If subset operation fails partway through
# DBSnapper provides recovery information
dbsnapper subset --resume target-name

# Or start fresh
dbsnapper subset --clean target-name
```

#### Data Validation
```bash
# Verify subset integrity after creation
dbsnapper dev validate-subset target-name

# Check foreign key constraints
dbsnapper dev assert-ts target-name --subset-validation
```

#### Rollback Procedures
```bash
# If subset results are unsatisfactory
# Drop and recreate destination database
dropdb subset_database
createdb subset_database

# Retry with modified configuration
dbsnapper subset target-name
```

## Advanced Configuration

### Complex WHERE Clauses
```yaml
subset_tables:
  # Date range with status filtering
  - table: orders
    where: |
      created_at BETWEEN '2024-01-01' AND '2024-12-31' 
      AND status IN ('completed', 'shipped', 'delivered')
      AND total_amount > 100.00

  # Subquery-based filtering
  - table: customers
    where: |
      customer_id IN (
        SELECT DISTINCT customer_id 
        FROM orders 
        WHERE created_at >= '2024-01-01'
      )

  # JSON field filtering (PostgreSQL)
  - table: products
    where: "metadata->>'category' = 'electronics'"
```

### Multi-Criteria Sampling
```yaml
subset_tables:
  # Different percentages by category
  - table: customers
    where: "customer_type = 'enterprise'"
    percent: 50
  - table: customers  
    where: "customer_type = 'individual'"
    percent: 10
```

### Relationship Mapping
```yaml
# Complex multi-column relationships
added_relationships:
  - fk_table: order_line_items
    fk_columns: ["order_id", "line_number"]
    ref_table: order_lines
    ref_columns: ["order_id", "line_number"]

# Cross-schema relationships
added_relationships:
  - fk_table: public.user_sessions
    fk_columns: user_id
    ref_table: auth.users
    ref_columns: id
```

## Integration Workflows

### CI/CD Integration
```bash
# Automated subset creation in CI pipeline
#!/bin/bash
set -e

echo "Creating test data subset..."
dbsnapper subset ci-test-data

echo "Loading into test database..."
dbsnapper load ci-test-environment

echo "Running integration tests..."
npm test -- --integration

echo "Cleanup test data..."
dbsnapper dev cleanup ci-test-environment
```

### Development Workflow
```bash
# Developer daily workflow
#!/bin/bash

# Update development database with fresh subset
dbsnapper subset dev-data-daily

# Load into local development
dbsnapper load local-dev-db

# Start development server
npm run dev
```

### QA Testing Workflow
```bash
# QA environment setup
#!/bin/bash

# Create consistent test dataset
dbsnapper subset qa-regression-data

# Load into QA environment
dbsnapper load qa-test-database

# Run automated test suite
pytest tests/ --database-url="postgres://localhost/qa-test-database"
```

## Troubleshooting

### Performance Issues

#### Slow Subset Creation
```
Issue: Subset operation takes too long
```

**Solutions:**
1. **Optimize WHERE clauses**: Use indexed columns
2. **Reduce subset size**: Lower percentages or add more restrictive WHERE clauses
3. **Check relationships**: Simplify complex relationship chains
4. **Database resources**: Ensure adequate CPU and memory
5. **Network latency**: Use local connections when possible

#### Memory Issues
```
Issue: Out of memory errors during processing
```

**Solutions:**
1. **Batch processing**: Enable batch mode for large operations
2. **Reduce subset size**: Process smaller subsets
3. **System resources**: Increase available system memory
4. **Database configuration**: Optimize database memory settings

### Data Integrity Issues

#### Missing Referenced Data
```
Issue: Foreign key constraint violations in subset
```

**Solutions:**
1. **Check configuration**: Verify all necessary relationships are defined
2. **Add missing tables**: Include referenced tables in copy_tables
3. **Relationship definition**: Add missing relationships to added_relationships
4. **Circular references**: Check for and resolve circular reference issues

#### Inconsistent Results
```
Issue: Subset results vary between runs
```

**Solutions:**
1. **Deterministic sampling**: Use consistent random seeds where possible
2. **WHERE clause specificity**: Make WHERE clauses more specific
3. **Data changes**: Account for source data changes between runs
4. **Configuration validation**: Verify configuration consistency

### Configuration Issues

#### Invalid Target Configuration
```
Issue: Subset configuration validation fails
```

**Solutions:**
1. **Required fields**: Ensure all required configuration fields are present
2. **URL validity**: Verify database connection URLs are correct
3. **Table names**: Check table names match database schema
4. **Syntax validation**: Validate YAML configuration syntax

#### Connection Problems
```
Issue: Cannot connect to source or destination database
```

**Solutions:**
1. **Connectivity**: Test database connections independently
2. **Permissions**: Verify database user permissions
3. **Network access**: Check network connectivity and firewalls
4. **Authentication**: Verify database credentials

### Diagnostic Commands
```bash
# Validate subset configuration
dbsnapper config validate --target subset-target

# Test database connections
dbsnapper dev test-connections subset-target

# Analyze database relationships
dbsnapper dev analyze-relationships subset-target

# Dry-run subset operation
dbsnapper subset subset-target --dry-run --verbose
```

!!! tip "Subset Planning"
    Start with small subsets and gradually increase size. Test subset configuration with small datasets before running on large production databases.

!!! note "Referential Integrity"
    DBSnapper automatically maintains referential integrity by following foreign key relationships. Manual relationship definition may be required for databases without proper constraints.

!!! warning "Production Impact"
    Subset operations read from source databases and may impact performance. Consider running during low-traffic periods and monitor database resources during operation.

## Related Commands

- [`build`](build.md) - Create database snapshots (can be combined with subset for smaller snapshots)
- [`load`](load.md) - Load subsets into destination databases
- [`sanitize`](sanitize.md) - Sanitize subset data for security and privacy
- [`targets`](targets.md) - List available targets with subset configurations

## See Also

- [Subset Introduction](../subset/introduction.md) - Complete overview of subsetting concepts
- [Subset Configuration](../subset/configuration.md) - Detailed configuration guide
