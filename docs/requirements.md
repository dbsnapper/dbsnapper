# Requirements

DBSnapper is designed to work across diverse environments, from local development machines to enterprise container orchestration platforms. Here's what you need to get started.

## System Requirements

### Supported Platforms

DBSnapper runs on all major platforms with pre-built binaries available from the [GitHub Releases](https://github.com/dbsnapper/dbsnapper/releases) page:

- **macOS** - Universal binary supporting both Intel (x86_64) and Apple Silicon (ARM64) architectures
- **Linux** - Full compatibility across distributions:
  - **x86_64** - Standard 64-bit Intel/AMD processors
  - **ARM64** - ARM 64-bit processors (AWS Graviton, Raspberry Pi 4+, etc.)
  - **Distributions**: Ubuntu, Debian, RHEL, CentOS, Fedora, Alpine Linux, and others

### Hardware Requirements

**Minimum System Requirements:**
- **CPU**: 1 core (multi-core recommended for large databases)
- **Memory**: 512MB RAM minimum, 2GB+ recommended
- **Storage**: 100MB for installation + space for snapshots
- **Network**: Internet access for DBSnapper Cloud integration (optional)

**Recommended for Production:**
- **CPU**: 4+ cores for optimal snapshot performance
- **Memory**: 4GB+ RAM for large database operations
- **Storage**: SSD storage for working directory
- **Network**: High-bandwidth connection for cloud storage operations

### Performance Scaling

DBSnapper v3.0 introduces **configurable CPU usage** for database operations:

```yaml
# Configure CPU usage globally
defaults:
  cpus: 4  # Use 4 CPU cores for all operations

# Override per target for large databases
targets:
  production_db:
    cpus: 8  # Use 8 cores for this target
```

**Performance Benefits:**
- **PostgreSQL**: Multi-core `pg_dump` and `pg_restore` operations
- **Large datasets**: Significantly faster snapshot creation and restoration
- **Concurrent operations**: Better resource utilization in container environments

## Database Support

### Supported Database Engines

DBSnapper leverages native database utilities for maximum compatibility and performance:

#### **PostgreSQL**
- **Versions**: PostgreSQL 10, 11, 12, 13, 14, 15, 16+
- **Deployment types**: Local, Docker, RDS, Aurora, Cloud SQL, self-hosted
- **Advanced features**:
  - Schema filtering (include/exclude specific schemas)
  - Multi-core parallel operations
  - Connection pooling support
  - SSL/TLS encryption

#### **MySQL**  
- **Versions**: MySQL 5.7, 8.0+, MariaDB 10.3+
- **Deployment types**: Local, Docker, RDS, Aurora, Cloud SQL, self-hosted
- **Features**:
  - Full database snapshots
  - Multi-core operations (when supported)
  - SSL/TLS encryption

### Database Access Requirements

**Source Database Permissions:**
- `SELECT` privilege on all tables to be snapshotted
- `SHOW DATABASES` privilege (MySQL) or connection access (PostgreSQL)
- For schema operations: appropriate schema-level permissions

**Destination Database Permissions:**
- `CREATE DATABASE` privilege on the destination server
- `DROP DATABASE` privilege (snapshots replace existing databases)
- Full read/write access to the target database

**Security Best Practices:**
- Use dedicated DBSnapper database users with minimal required permissions
- Enable SSL/TLS connections for production databases
- Consider read replicas for snapshot sources to reduce production load

## Database Client Tools

DBSnapper requires database client tools for snapshot operations. You have two options:

### Option 1: Docker-Based Tools (Recommended)

Use the DBSnapper Docker image with all tools pre-installed:

```bash
docker pull ghcr.io/dbsnapper/dbsnapper:latest
```

**Benefits:**
- All tools included and version-tested
- Consistent environment across different systems
- No local tool installation required
- Perfect for CI/CD pipelines and containers

### Option 2: Local Tools Installation

Install database client tools on your local system:

=== "macOS"
    ```bash
    # PostgreSQL tools
    brew install postgresql
    
    # MySQL tools
    brew install mysql-client
    ```

=== "Ubuntu/Debian"
    ```bash
    # PostgreSQL tools
    sudo apt-get install postgresql-client
    
    # MySQL tools
    sudo apt-get install mysql-client
    ```

=== "RHEL/CentOS/Fedora"
    ```bash
    # PostgreSQL tools
    sudo dnf install postgresql
    
    # MySQL tools
    sudo dnf install mysql
    ```

=== "Alpine Linux"
    ```bash
    # PostgreSQL tools
    sudo apk add postgresql-client
    
    # MySQL tools
    sudo apk add mysql-client
    ```

**Required Tools:**
- **PostgreSQL**: `psql`, `pg_dump`, `pg_restore`
- **MySQL**: `mysql`, `mysqldump`

## Container and Orchestration Support

### Docker Requirements

- **Docker Engine**: Version 18.09+ recommended
- **Docker permissions**: User must be able to run Docker containers
- **Network access**: Container networking for database connections

### Kubernetes/Container Orchestration

DBSnapper works excellently in container orchestration platforms:

**Resource Requirements:**
```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "250m"
  limits:
    memory: "2Gi"
    cpu: "2000m"  # Adjust based on CPU configuration
```

**Volume Requirements:**
- Configuration volume: `~/.config/dbsnapper`
- Working directory volume: `~/.dbsnapper` (for local snapshots)
- Optional: Shared storage for team access

### CI/CD Integration

DBSnapper integrates seamlessly with CI/CD pipelines:

**GitHub Actions:**
```yaml
- uses: dbsnapper/install-dbsnapper-agent-action@v1
  with:
    version: latest
```

**GitLab CI, Jenkins, etc.:** Use the Docker image for consistent environments

## Cloud Storage Integration

### Supported Storage Providers

DBSnapper follows a **"Bring Your Own Cloud Storage"** model for maximum security and flexibility:

#### **Amazon S3**
- AWS S3 (all regions)
- S3-compatible services (MinIO, DigitalOcean Spaces, etc.)
- **Authentication**: AWS CLI profiles, IAM roles, access keys
- **Features**: Server-side encryption, versioning, lifecycle policies

#### **Cloudflare R2**
- Cloudflare R2 storage
- **Authentication**: API tokens, S3-compatible credentials
- **Benefits**: Zero egress fees, global distribution

#### **Additional S3-Compatible Providers**
- **MinIO**: Self-hosted object storage
- **DigitalOcean Spaces**: Managed object storage
- **Backblaze B2**: Cost-effective cloud storage
- **Wasabi**: Hot cloud storage

### Storage Requirements

**Capacity Planning:**
- Snapshot sizes typically 10-50% of original database size (due to compression)
- Plan for multiple snapshots per target (original + sanitized versions)
- Consider retention policies for automated cleanup

**Performance Considerations:**
- High-bandwidth network connections for large database uploads
- Consider storage regions close to your database infrastructure
- Use presigned URLs for secure, direct uploads/downloads

## Network and Security Requirements

### Network Access

**Outbound connections required for:**
- Database servers (source and destination)
- DBSnapper Cloud API (if using cloud features): `app.dbsnapper.com`
- Cloud storage providers (S3, R2, etc.)
- Container registry: `ghcr.io` (for Docker images)

**Firewall Considerations:**
- Standard database ports: PostgreSQL (5432), MySQL (3306)
- HTTPS (443) for cloud API and storage operations
- Custom ports if using non-standard database configurations

### Security Requirements

**Encryption:**
- **In transit**: TLS 1.2+ for all network communications
- **At rest**: AES-256 encryption for stored configurations and snapshots
- **Database connections**: SSL/TLS supported and recommended

**Authentication:**
- Support for all standard database authentication methods
- Integration with AWS IAM, OIDC, and other identity providers
- Optional: SSO integration with Okta, Auth0, Azure AD

## Development Environment Integration

### IDE Support

**VS Code Extension**: [DBSnapper for VS Code](https://marketplace.visualstudio.com/items?itemName=dbsnapper.vscode-dbsnapper)
- In-editor snapshot management
- Target configuration and testing
- Direct integration with workspace settings

### MCP Server Integration

**AI Assistant Integration:**
- Model Context Protocol (MCP) server for Claude Desktop and compatible AI assistants
- Programmatic database operations through AI conversation
- Requires DBSnapper installation and configuration

### Terraform Provider

**Infrastructure as Code**: [DBSnapper Terraform Provider](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest)
- Manage DBSnapper resources with Terraform
- Integration with existing infrastructure automation
- Supports all major DBSnapper configuration options

## Compatibility Matrix

| Feature | Local Mode | Cloud Mode | Docker | Kubernetes | CI/CD |
|---------|------------|------------|--------|------------|-------|
| PostgreSQL Support | ✅ | ✅ | ✅ | ✅ | ✅ |
| MySQL Support | ✅ | ✅ | ✅ | ✅ | ✅ |
| Schema Filtering | ✅ | ✅ | ✅ | ✅ | ✅ |
| Multi-Core Operations | ✅ | ✅ | ✅ | ✅ | ✅ |
| Cloud Storage | ❌ | ✅ | ✅ | ✅ | ✅ |
| Team Sharing | ❌ | ✅ | ✅ | ✅ | ✅ |
| SSO Integration | ❌ | ✅ | ✅ | ✅ | ✅ |
| MCP Server | ✅ | ✅ | ✅ | ✅ | ❌ |

## Getting Started

With these requirements in mind, choose your deployment approach:

1. **Quick Trial**: Use the [Docker image](installation.md) - no local setup required
2. **Local Development**: [Install natively](installation.md) for your platform  
3. **Team Setup**: [Configure DBSnapper Cloud](https://app.dbsnapper.com/sign_up) for collaboration
4. **Production Deployment**: Use [container orchestration](installation.md#container-orchestration) for scale

**Next:** Ready to install? Check out our **[Installation Guide](installation.md)** for step-by-step instructions.