# Auth Command

The `auth` command manages authentication with DBSnapper Cloud, enabling access to cloud features including shared targets, cloud storage, and team collaboration capabilities. This command serves as the gateway to DBSnapper's cloud ecosystem.

## Overview

The auth command provides cloud authentication management:
- **Cloud connectivity**: Enables access to DBSnapper Cloud services
- **API token management**: Secure storage and configuration of authentication tokens
- **Team collaboration**: Unlocks shared targets and team snapshot sharing
- **Cloud storage**: Enables cloud storage profiles for snapshot backup and sharing
- **SSO integration**: Facilitates Okta SSO-based target sharing

## Syntax

```bash
dbsnapper auth [command]
```

## Available Subcommands

### Authentication Commands

| Command | Purpose | Use When |
|---------|---------|----------|
| [`token`](auth-token.md) | Add DBSnapper Cloud API token | Setting up cloud integration |

## Cloud Integration Benefits

### Core Cloud Features
Once authenticated, DBSnapper Cloud provides:

#### Cloud Targets
- **Centralized management**: Manage targets via web interface
- **Team synchronization**: Targets automatically sync across team members
- **Configuration backup**: Target configurations stored securely in cloud
- **Cross-device access**: Access same targets from multiple machines

#### Cloud Storage Profiles
- **S3-compatible storage**: Support for AWS S3, Cloudflare R2, and other providers
- **Snapshot backup**: Automatic snapshot backup to cloud storage
- **Cross-environment sharing**: Share snapshots between development, staging, and production
- **Disaster recovery**: Cloud-based snapshot recovery capabilities

#### Shared Targets
- **Okta SSO integration**: Share targets based on Okta group membership
- **Secure sharing**: Only sanitized snapshots are shared for security
- **Access control**: Fine-grained control over who can access shared data
- **Audit trail**: Track access to shared snapshots

#### Team Collaboration
- **Centralized configuration**: Share target configurations across team
- **Collaborative development**: Team members can access shared development data
- **Environment consistency**: Ensure consistent data across team environments
- **Knowledge sharing**: Share database schemas and structures

## Authentication Workflow

### Initial Cloud Setup
```bash
# 1. Register account at DBSnapper Cloud
# Visit https://app.dbsnapper.com and create account

# 2. Get API token from cloud dashboard
# Navigate to Get Started page in DBSnapper Cloud app

# 3. Configure authentication
dbsnapper auth token <your_api_token>

# 4. Verify cloud connectivity
dbsnapper targets  # Should now show cloud targets
```

### Cloud Features Verification
```bash
# Test cloud connectivity
dbsnapper targets | grep cloud

# Check for shared targets
dbsnapper targets | grep share

# Verify cloud storage capabilities
dbsnapper config check  # Should show "DBSnapper Cloud connected"
```

## Cloud vs Local Mode

### Local Mode (No Authentication)
**Available Features:**
- Local target management
- Local snapshot storage
- Local configuration only
- Standalone operation

**Limitations:**
- No team collaboration
- No cloud storage backup
- No shared targets
- No cross-device synchronization

### Cloud Mode (Authenticated)
**Additional Features:**
- All local mode features, plus:
- Cloud target management
- Cloud storage profiles
- Shared targets via Okta SSO
- Team collaboration
- Cross-device synchronization
- Web-based configuration management

## Security Considerations

### API Token Security
- **Secure storage**: Tokens stored encrypted in configuration file
- **Limited scope**: Tokens have limited permissions and scope
- **Rotation**: Tokens can be rotated regularly for security
- **Revocation**: Tokens can be revoked from cloud dashboard

### Data Protection
- **Encrypted transit**: All cloud communications use TLS encryption
- **Data sovereignty**: Choose cloud storage regions for compliance
- **Access controls**: Fine-grained permissions and access controls
- **Audit logging**: Complete audit trail of cloud operations

### Best Practices
```bash
# Use environment variables for tokens in automation
export DBSNAPPER_AUTHTOKEN="your_token_here"

# Rotate tokens regularly
# 1. Generate new token in cloud dashboard
# 2. Update configuration: dbsnapper auth token <new_token>
# 3. Revoke old token in cloud dashboard

# Separate tokens for different environments
dbsnapper auth token --config dev.yml <dev_token>
dbsnapper auth token --config prod.yml <prod_token>
```

## Example Usage

### Basic Authentication Setup
```bash
# Set up cloud authentication
dbsnapper auth token hNsdelkjcxgoiuwalkmcgoi...

# Verify authentication worked
dbsnapper targets  # Should include cloud targets
```

### Environment-Specific Authentication
```bash
# Set up different tokens for different environments
dbsnapper auth token --config ~/.config/dbsnapper/dev.yml <dev_token>
dbsnapper auth token --config ~/.config/dbsnapper/prod.yml <prod_token>

# Use specific environment
dbsnapper --config ~/.config/dbsnapper/dev.yml targets
```

### Team Setup Workflow
```bash
# Team lead sets up cloud integration
dbsnapper auth token <team_lead_token>

# Create and share targets via cloud interface
# (Done through DBSnapper Cloud web app)

# Team members authenticate with their tokens
dbsnapper auth token <team_member_token>

# Verify access to shared targets
dbsnapper targets | grep share
```

## Use Cases

### Individual Developer Setup
```bash
# Personal cloud backup and multi-device access
dbsnapper auth token <personal_token>

# Configure cloud storage for snapshot backup
# (Edit configuration to add storage profiles)

# Access targets from multiple development machines
```

### Team Collaboration
```bash
# Development team sharing sanitized production data
# 1. Team lead configures production targets in cloud
# 2. Team members authenticate with their cloud accounts
dbsnapper auth token <team_member_token>

# 3. Access shared sanitized snapshots
dbsnapper targets | grep share
dbsnapper pull shared-production-data
```

### Multi-Environment Management
```bash
# Manage development, staging, and production environments
# Each environment uses separate cloud accounts/tokens

# Development environment
dbsnapper auth token --config dev.yml <dev_token>

# Staging environment  
dbsnapper auth token --config staging.yml <staging_token>

# Production environment
dbsnapper auth token --config prod.yml <prod_token>
```

### CI/CD Pipeline Integration
```bash
# Automated pipeline with cloud integration
export DBSNAPPER_AUTHTOKEN="$CI_DBSNAPPER_TOKEN"

# Use cloud targets in automated testing
dbsnapper pull shared-test-data
dbsnapper load ci-test-environment
```

### Disaster Recovery Setup
```bash
# Set up cloud storage for disaster recovery
dbsnapper auth token <recovery_token>

# Configure cloud storage profiles for backup
# (Edit configuration to add S3-compatible storage)

# Regular backup to cloud
dbsnapper build critical-database  # Automatically backs up to cloud
```

## Cloud Service Integration

### DBSnapper Cloud Web Interface
- **Target management**: Create and configure targets via web interface
- **Storage profiles**: Configure cloud storage providers
- **Team management**: Invite team members and manage permissions
- **Snapshot browser**: Browse and manage cloud snapshots
- **Usage monitoring**: Monitor snapshot usage and costs

### Okta SSO Integration
- **Enterprise authentication**: Use existing Okta accounts
- **Group-based sharing**: Share targets based on Okta group membership
- **Automatic provisioning**: Automatic access provisioning/deprovisioning
- **Compliance**: Maintain compliance with enterprise security policies

### Cloud Storage Providers
- **AWS S3**: Native integration with Amazon S3
- **Cloudflare R2**: Cost-effective S3-compatible storage
- **Custom S3**: Any S3-compatible storage provider
- **Multi-region**: Support for multiple storage regions

## Troubleshooting Authentication

### Common Issues

#### Invalid Token
```bash
Error: authentication failed - invalid token
```

**Solutions:**
1. Verify token is correct (check for typos)
2. Generate new token from cloud dashboard
3. Check token hasn't expired or been revoked

#### Network Connectivity
```bash
Error: cannot connect to DBSnapper Cloud
```

**Solutions:**
1. Check internet connectivity
2. Verify DNS resolution
3. Check corporate firewall settings
4. Try different network connection

#### Configuration Issues
```bash
Error: configuration file not found
```

**Solutions:**
1. Initialize configuration: `dbsnapper config init`
2. Specify config file path: `--config /path/to/config.yml`
3. Verify file permissions

### Diagnostic Commands
```bash
# Test cloud connectivity
dbsnapper config check | grep "DBSnapper Cloud"

# Verify authentication
dbsnapper targets | grep -E "(cloud|share)"

# Check configuration
cat ~/.config/dbsnapper/dbsnapper.yml | grep authtoken
```

## Integration with Other Commands

### Cloud-Dependent Operations
These commands require authentication for full functionality:

```bash
# Access cloud and shared targets
dbsnapper targets

# Pull cloud snapshots
dbsnapper pull cloud-target-name

# Build with cloud storage backup
dbsnapper build production-db  # Uploads to cloud if storage profile configured
```

### Configuration Integration
```bash
# Authentication works with all config commands
dbsnapper config check  # Shows cloud connectivity status
dbsnapper config init   # Can be run before or after authentication
```

## Migration from Local to Cloud

### Gradual Migration Strategy
```bash
# 1. Start with local configuration
dbsnapper config init

# 2. Add cloud authentication
dbsnapper auth token <your_token>

# 3. Verify cloud connectivity
dbsnapper config check

# 4. Gradually migrate targets to cloud management
# (Use web interface to recreate targets in cloud)

# 5. Configure cloud storage for existing targets
# (Edit configuration to add storage profiles)
```

### Hybrid Mode
You can operate in hybrid mode with both local and cloud targets:

```bash
# List all targets (local, cloud, and shared)
dbsnapper targets

# Use local targets normally
dbsnapper build local-dev-db

# Use cloud targets with full cloud features
dbsnapper build cloud-prod-db
dbsnapper pull shared-team-data
```

## Best Practices

### Authentication Management
1. **Secure storage**: Protect configuration files containing tokens
2. **Token rotation**: Regularly rotate authentication tokens
3. **Environment separation**: Use separate tokens for different environments
4. **Access monitoring**: Monitor token usage and access patterns

### Team Coordination
1. **Onboarding**: Include cloud setup in team onboarding process
2. **Documentation**: Document cloud setup procedures for team
3. **Access review**: Regularly review team member access
4. **Training**: Train team on cloud features and security practices

### Security Operations
1. **Principle of least privilege**: Grant minimum required permissions
2. **Regular audits**: Audit cloud access and usage regularly
3. **Incident response**: Have procedures for token compromise
4. **Compliance**: Ensure cloud usage meets compliance requirements

!!! tip "Cloud Benefits"
    Authenticating with DBSnapper Cloud unlocks powerful team collaboration features including shared targets, cloud storage backup, and cross-environment snapshot sharing.

!!! note "Token Security"
    API tokens are stored encrypted in your configuration file. Protect your configuration file and rotate tokens regularly for security.

!!! warning "Network Requirements"
    Cloud features require internet connectivity. Some corporate networks may require firewall configuration to allow access to DBSnapper Cloud services.

## Related Commands

- [`auth token`](auth-token.md) - Configure API token for cloud authentication
- [`targets`](targets.md) - List targets (shows cloud and shared targets when authenticated)
- [`pull`](pull.md) - Pull cloud snapshots (requires authentication)
- [`config check`](config-check.md) - Verify cloud connectivity and authentication

## See Also

- [DBSnapper Cloud](../dbsnapper-cloud/introduction.md) - Complete cloud features overview
- [Shared Targets](../dbsnapper-cloud/targets.md) - Team collaboration with shared targets
- [Storage Profiles](../dbsnapper-cloud/storage_profiles.md) - Cloud storage configuration
- [SSO Integration](../dbsnapper-cloud/sso/index.md) - Okta SSO setup and configuration