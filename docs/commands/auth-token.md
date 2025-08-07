# Auth Token Command

The `auth token` command configures your DBSnapper Cloud API token, enabling authentication with DBSnapper Cloud services. This essential command unlocks cloud features including shared targets, cloud storage, and team collaboration capabilities.

## Overview

The auth token command provides secure API token management:
- **Cloud authentication**: Establishes secure connection to DBSnapper Cloud
- **Token storage**: Encrypts and stores API tokens in configuration file
- **Access enablement**: Unlocks cloud targets, shared targets, and cloud storage
- **Team integration**: Enables participation in team collaboration features
- **Multi-environment support**: Manages different tokens for different environments

## Syntax

```bash
dbsnapper auth token <api_token> [flags]
```

## Arguments

- `<api_token>`: The API token obtained from DBSnapper Cloud dashboard

## Options

```bash
  -h, --help   help for token
```

## API Token Acquisition

### Getting Your API Token

1. **Visit DBSnapper Cloud**: Navigate to [https://app.dbsnapper.com](https://app.dbsnapper.com)
2. **Create Account**: Register for a new account or sign in to existing account
3. **Access Dashboard**: Navigate to the main dashboard after login
4. **Get Started Page**: Go to the "Get Started" section
5. **Copy Token**: Copy your personal API token from the Get Started page

### Token Characteristics
- **Unique identifier**: Each user has a unique API token
- **Secure format**: Long, cryptographically secure string
- **Limited scope**: Token permissions are limited to your account
- **Revocable**: Tokens can be revoked from the cloud dashboard

## How It Works

### Token Storage Process
1. **Validation**: Verifies token format and basic validity
2. **Encryption**: Encrypts token using your configuration's secret key
3. **Configuration update**: Adds encrypted token to configuration file
4. **Cloud connection**: Tests connection to verify token functionality
5. **Feature activation**: Enables cloud features for subsequent commands

### Configuration Integration
The token is stored in your configuration file's `authtoken` field:

```yaml
# Before token configuration
authtoken: ""

# After token configuration
authtoken: "encrypted_token_value_here"
```

## Token Security

### Encryption at Rest
- **Secret key encryption**: Tokens encrypted using your configuration's secret key
- **No plain text**: Tokens never stored in plain text
- **File protection**: Configuration file should have restricted permissions

### Secure Handling
```bash
# Set appropriate file permissions
chmod 600 ~/.config/dbsnapper/dbsnapper.yml

# Verify permissions
ls -la ~/.config/dbsnapper/dbsnapper.yml
# Should show: -rw------- (owner read/write only)
```

### Environment Variable Alternative
For automation and CI/CD, use environment variables:

```bash
# Set token via environment variable
export DBSNAPPER_AUTHTOKEN="your_token_here"

# DBSnapper will use environment variable over config file
dbsnapper targets  # Uses token from environment
```

## Example Usage

### Basic Token Configuration
```bash
# Configure API token (replace with your actual token)
dbsnapper auth token hNsdelkjcxgoiuwalkmcgoi1234567890abcdef
```

**Output:**
```
✅ DBSnapper Cloud API token configured successfully
✅ Cloud connectivity verified
🔗 You now have access to cloud features:
  - Cloud targets
  - Shared targets  
  - Cloud storage profiles
  - Team collaboration
```

### Environment-Specific Configuration
```bash
# Configure token for development environment
dbsnapper auth token --config ~/.config/dbsnapper/dev.yml <dev_token>

# Configure token for production environment
dbsnapper auth token --config ~/.config/dbsnapper/prod.yml <prod_token>

# Use specific environment
dbsnapper --config ~/.config/dbsnapper/dev.yml targets
```

### Verification After Configuration
```bash
# Verify cloud connectivity
dbsnapper config check | grep "DBSnapper Cloud"

# List targets to see cloud and shared targets
dbsnapper targets

# Check for cloud-specific features
dbsnapper targets | grep -E "(cloud|share)"
```

## Use Cases

### Individual Developer Setup
```bash
# Personal development setup
dbsnapper auth token <personal_token>

# Verify access to personal cloud targets
dbsnapper targets | grep cloud

# Access shared team data
dbsnapper targets | grep share
```

### Team Member Onboarding
```bash
# New team member setup
# 1. Team lead provides onboarding instructions
# 2. New member creates DBSnapper Cloud account
# 3. Configures authentication
dbsnapper auth token <new_member_token>

# 4. Verifies access to shared team resources
dbsnapper targets | grep share
dbsnapper pull shared-development-data
```

### CI/CD Pipeline Configuration
```bash
# Automated pipeline setup
# Store token in CI/CD secrets management
export DBSNAPPER_AUTHTOKEN="$CI_DBSNAPPER_TOKEN"

# Verify authentication in pipeline
if ! dbsnapper config check | grep -q "DBSnapper Cloud connected"; then
    echo "Cloud authentication failed"
    exit 1
fi

# Use cloud features in automation
dbsnapper pull production-snapshots
```

### Multi-Environment Management
```bash
# Development environment
dbsnapper auth token --config config/dev.yml <dev_token>

# Staging environment
dbsnapper auth token --config config/staging.yml <staging_token>

# Production environment
dbsnapper auth token --config config/prod.yml <prod_token>

# Verify each environment
for env in dev staging prod; do
    echo "Testing $env environment..."
    dbsnapper --config config/$env.yml config check
done
```

### Token Rotation
```bash
# Regular security maintenance
# 1. Generate new token in cloud dashboard
# 2. Update configuration
dbsnapper auth token <new_token>

# 3. Verify new token works
dbsnapper config check

# 4. Revoke old token in cloud dashboard
```

## Cloud Features Enabled

### After Token Configuration
Once authenticated, you gain access to:

#### Cloud Targets
```bash
# List cloud-managed targets
dbsnapper targets | grep cloud

# Build snapshots with cloud storage
dbsnapper build cloud-production-db
```

#### Shared Targets
```bash
# Access team-shared targets (via Okta SSO)
dbsnapper targets | grep share

# Pull shared sanitized snapshots
dbsnapper pull team-development-data
```

#### Cloud Storage Profiles
```yaml
# Configure cloud storage in your configuration
storage_profiles:
  aws-s3:
    type: "s3"
    region: "us-west-2"
    bucket: "my-snapshots"
    access_key_id: "${AWS_ACCESS_KEY_ID}"
    secret_access_key: "${AWS_SECRET_ACCESS_KEY}"
```

#### Web Interface Integration
- **Target management**: Manage targets via web interface
- **Snapshot browsing**: Browse snapshots in cloud dashboard
- **Team management**: Manage team members and permissions
- **Usage monitoring**: Monitor snapshot usage and costs

## Token Management

### Token Lifecycle
1. **Generation**: Create token in DBSnapper Cloud dashboard
2. **Configuration**: Add token to DBSnapper configuration
3. **Usage**: Token enables cloud features
4. **Rotation**: Regularly rotate tokens for security
5. **Revocation**: Revoke tokens when no longer needed

### Best Practices
```bash
# Regular token rotation (monthly/quarterly)
# 1. Generate new token
# 2. Test new token in non-production first
dbsnapper auth token --config test.yml <new_token>
dbsnapper --config test.yml config check

# 3. Update production configuration
dbsnapper auth token <new_token>

# 4. Revoke old token
```

### Multiple Token Management
```bash
# Different tokens for different purposes
dbsnapper auth token --config personal.yml <personal_token>
dbsnapper auth token --config work.yml <work_token>
dbsnapper auth token --config project-a.yml <project_a_token>

# Use appropriate configuration for each context
alias dbsnapper-personal='dbsnapper --config personal.yml'
alias dbsnapper-work='dbsnapper --config work.yml'
```

## Troubleshooting

### Common Issues

#### Invalid Token Format
```bash
Error: invalid token format
```

**Solutions:**
1. Verify token was copied completely from cloud dashboard
2. Check for extra spaces or characters
3. Regenerate token from cloud dashboard if needed

#### Token Authentication Failed
```bash
Error: authentication failed - invalid token
```

**Solutions:**
1. Verify token is current and not revoked
2. Check network connectivity to DBSnapper Cloud
3. Generate new token from cloud dashboard
4. Verify cloud service availability

#### Network Connection Issues
```bash
Error: cannot connect to DBSnapper Cloud
```

**Solutions:**
1. Check internet connectivity
2. Verify DNS resolution
3. Check corporate firewall settings
4. Try different network connection
5. Contact IT about cloud service access

#### Configuration File Issues
```bash
Error: cannot write to configuration file
```

**Solutions:**
```bash
# Check file permissions
ls -la ~/.config/dbsnapper/dbsnapper.yml

# Fix permissions if needed
chmod 600 ~/.config/dbsnapper/dbsnapper.yml

# Check directory permissions
ls -la ~/.config/dbsnapper/

# Create directory if missing
mkdir -p ~/.config/dbsnapper
```

### Diagnostic Commands
```bash
# Test cloud connectivity
dbsnapper config check

# Verify token is configured
grep authtoken ~/.config/dbsnapper/dbsnapper.yml

# Test cloud features
dbsnapper targets | grep -E "(cloud|share)"

# Check environment variable
echo $DBSNAPPER_AUTHTOKEN
```

## Security Considerations

### Token Protection
1. **File permissions**: Restrict configuration file access
2. **Version control**: Never commit tokens to version control
3. **Environment variables**: Use environment variables in automation
4. **Regular rotation**: Rotate tokens regularly
5. **Revocation**: Revoke compromised tokens immediately

### Access Control
1. **Principle of least privilege**: Use tokens with minimal required permissions
2. **Separate tokens**: Use different tokens for different environments
3. **Team management**: Regularly review team member access
4. **Audit logging**: Monitor token usage and access patterns

### Incident Response
```bash
# If token is compromised:
# 1. Revoke token immediately in cloud dashboard
# 2. Generate new token
# 3. Update all configurations
dbsnapper auth token <new_secure_token>

# 4. Verify security
dbsnapper config check

# 5. Review access logs in cloud dashboard
```

## Integration with Other Commands

### Cloud-Enabled Commands
After token configuration, these commands gain cloud functionality:

```bash
# Enhanced target listing (includes cloud and shared targets)
dbsnapper targets

# Cloud snapshot operations
dbsnapper pull cloud-target-name
dbsnapper build cloud-target-name  # May upload to cloud storage

# Configuration checking includes cloud connectivity
dbsnapper config check
```

### Workflow Integration
```bash
# Complete cloud setup workflow
dbsnapper config init              # Initialize configuration
dbsnapper auth token <your_token>  # Configure authentication
dbsnapper config check             # Verify cloud connectivity
dbsnapper targets                  # List available targets
```

## Environment Variables

### Alternative Token Configuration
```bash
# Set token via environment variable
export DBSNAPPER_AUTHTOKEN="your_token_here"

# Verify environment variable is used
dbsnapper config check | grep "DBSnapper Cloud"
```

### CI/CD Integration
```yaml
# GitHub Actions example
env:
  DBSNAPPER_AUTHTOKEN: ${{ secrets.DBSNAPPER_TOKEN }}

steps:
  - name: Test DBSnapper Cloud connectivity
    run: |
      dbsnapper config check
      dbsnapper targets | grep cloud
```

## Best Practices Summary

### Security Best Practices
1. **Secure storage**: Use file permissions and encryption
2. **Environment separation**: Different tokens per environment
3. **Regular rotation**: Rotate tokens periodically
4. **Access monitoring**: Monitor token usage and access
5. **Incident response**: Have procedures for compromised tokens

### Operational Best Practices
1. **Team onboarding**: Include token setup in onboarding process
2. **Documentation**: Document token management procedures
3. **Testing**: Test tokens in non-production environments first
4. **Backup access**: Maintain alternative access methods
5. **Compliance**: Ensure token usage meets compliance requirements

!!! tip "Cloud Dashboard"
    Generate and manage your API tokens through the DBSnapper Cloud dashboard at https://app.dbsnapper.com. The Get Started page provides your current API token.

!!! note "Environment Variables"
    For automation and CI/CD, use the `DBSNAPPER_AUTHTOKEN` environment variable instead of storing tokens in configuration files.

!!! warning "Token Security"
    Protect your API tokens like passwords. Use appropriate file permissions, rotate regularly, and never commit tokens to version control.

## Related Commands

- [`auth`](auth.md) - Overview of authentication commands and cloud integration
- [`targets`](targets.md) - List targets (shows cloud and shared targets after authentication)
- [`pull`](pull.md) - Pull cloud snapshots (requires authentication)
- [`config check`](config-check.md) - Verify cloud connectivity and authentication status

## See Also

- [DBSnapper Cloud](../dbsnapper-cloud/introduction.md) - Complete overview of cloud features
- [Shared Targets](../dbsnapper-cloud/targets.md) - Team collaboration with shared targets
- [Storage Profiles](../dbsnapper-cloud/storage_profiles.md) - Cloud storage configuration
- [SSO Integration](../dbsnapper-cloud/sso/index.md) - Enterprise SSO setup and configuration