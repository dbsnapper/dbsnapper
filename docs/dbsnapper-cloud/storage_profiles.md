---
title: Storage Profiles - Cloud Storage Configuration
description: Configure secure cloud storage for your database snapshots with comprehensive support for S3, R2, and other providers. Complete guide to setup, security, and best practices.
---

# Storage Profiles: Secure Cloud Storage for Database Snapshots

Storage Profiles define where and how your database snapshots are stored in cloud storage providers. They're essential for team collaboration, backup strategies, and implementing proper security boundaries between raw and sanitized data.

## Why Cloud Storage for Database Snapshots?

### **🔗 Team Collaboration**
**Enable seamless snapshot sharing across your organization:**
- Developers can access the latest sanitized snapshots instantly
- QA teams get consistent test data for reliable bug reproduction  
- CI/CD pipelines automatically use up-to-date snapshots

### **🛡️ Security Boundaries**
**Implement proper data access controls:**
- **Raw snapshots** → Restricted storage (production team access only)
- **Sanitized snapshots** → Shared storage (broader team access)
- **Audit trails** → Track all snapshot access and modifications

### **💰 Cost Efficiency**
**Optimize storage costs and performance:**
- Compressed snapshots typically use 10-50% of original database size
- Lifecycle policies automatically manage snapshot retention
- Multi-region storage for disaster recovery scenarios

## Supported Cloud Storage Providers

DBSnapper follows a **"Bring Your Own Cloud Storage"** approach—use your existing cloud storage accounts with your security policies and access controls.

=== "**🌐 Amazon S3**"

    **Full S3 ecosystem support:**
    ```yaml
    storage_profiles:
      production_s3:
        provider: s3
        region: us-west-2
        bucket: dbsnapper-prod-snapshots
        prefix: raw-snapshots
        awscli_profile: production  # Use AWS CLI credentials
        
      sanitized_s3:
        provider: s3
        region: us-west-2
        bucket: dbsnapper-team-snapshots
        prefix: sanitized-snapshots  
        awscli_profile: team-access  # Different credentials for team access
    ```

    **Benefits:**
    - Integrated with your existing AWS IAM policies
    - Server-side encryption (SSE-S3, SSE-KMS)
    - Intelligent tiering for cost optimization
    - Cross-region replication for disaster recovery

=== "**⚡ Cloudflare R2**"

    **Zero egress fees with global performance:**
    ```yaml
    storage_profiles:
      r2_sanitized:
        provider: r2
        bucket: dbsnapper-r2-sanitized
        prefix: team-snapshots
        account_id: your-cloudflare-account-id
        awscli_profile: r2_profile  # Configure AWS CLI for R2
    ```

    **Benefits:**
    - Zero egress charges (significant cost savings)
    - Global edge network for fast access
    - S3-compatible API (works with existing tools)
    - Built-in DDoS protection

=== "**🏠 MinIO (Self-Hosted)**"

    **Complete control with on-premises or private cloud:**
    ```yaml
    storage_profiles:
      minio_local:
        provider: minio
        endpoint: https://minio.yourcompany.com
        bucket: dbsnapper-snapshots
        prefix: production
        access_key: your-minio-access-key
        secret_key: your-minio-secret-key
    ```

    **Benefits:**
    - Complete data sovereignty
    - No vendor lock-in
    - Kubernetes-native deployments
    - Full control over security and compliance

=== "**🌊 DigitalOcean Spaces**"

    **Simple, cost-effective object storage:**
    ```yaml  
    storage_profiles:
      do_spaces:
        provider: dospaces
        endpoint: https://nyc3.digitaloceanspaces.com
        bucket: dbsnapper-do-snapshots
        prefix: team-data
        access_key: your-spaces-key
        secret_key: your-spaces-secret
    ```

## Creating Storage Profiles

### **Option 1: DBSnapper Cloud Web Interface**

**Visual Configuration (Recommended for most users):**

1. **Navigate to Storage Profiles** in DBSnapper Cloud
2. **Click "New Storage Profile"** to create a profile
3. **Configure Provider Settings** using the forms below
4. **Test connectivity** before saving

![DBSnapper Cloud - Storage Profiles](../static/cloud/storage-profiles-index.png "Storage Profiles management interface")

### **Option 2: Local Configuration File**

**Advanced users can define storage profiles in their local configuration:**

```yaml title="~/.config/dbsnapper/dbsnapper.yml"
# Define storage profiles locally (synced to cloud when authtoken present)
storage_profiles:
  prod_s3:
    provider: s3
    region: us-west-2
    bucket: mycompany-db-snapshots
    prefix: production
    awscli_profile: dbsnapper-prod
    
  team_r2:
    provider: r2
    bucket: team-sanitized-data
    prefix: shared
    account_id: your-account-id
    awscli_profile: r2_team

# Use in targets
targets:
  production_api:
    storage_profile: prod_s3        # Raw snapshots go here
    sanitize:
      storage_profile: team_r2      # Sanitized snapshots shared here
```

## Storage Profile Configuration

### **Basic Configuration**

**Profile Name**
: Unique identifier for this storage profile (used in target configuration)

**Provider**
: Cloud storage provider (s3, r2, minio, dospaces)

**Bucket**
: Storage bucket/container name where snapshots will be stored

**Prefix** _(Optional)_
: Folder/path prefix for organizing snapshots (e.g., `production/`, `team-snapshots/`)

### **Authentication Methods**

=== "**🔑 AWS CLI Profiles (Recommended)**"

    **Most secure approach using your existing AWS CLI configuration:**

    ```bash
    # Configure AWS CLI profile for DBSnapper
    aws configure --profile dbsnapper-prod
    
    # Configure additional profile for R2
    aws configure --profile r2_team \
      --endpoint-url https://your-account-id.r2.cloudflarestorage.com
    ```

    ```yaml
    storage_profiles:
      secure_s3:
        provider: s3
        bucket: secure-snapshots
        awscli_profile: dbsnapper-prod  # Uses existing CLI credentials
    ```

    **Benefits:**
    - Credentials stored in standard AWS CLI location
    - Supports MFA and role assumption
    - Works with AWS SSO and temporary credentials
    - No credentials in DBSnapper configuration files

=== "**🔐 Direct Credentials**"

    **For environments where AWS CLI profiles aren't available:**

    ```yaml
    storage_profiles:
      direct_s3:
        provider: s3
        region: us-east-1
        bucket: dbsnapper-direct
        access_key: AKIA...
        secret_key: xyz...
    ```

    !!! warning "Security Consideration"
        Store sensitive credentials as environment variables rather than in configuration files:
        ```yaml
        storage_profiles:
          secure_direct:
            provider: s3
            bucket: dbsnapper-secure
            access_key: "{{`AWS_ACCESS_KEY_ID` | env}}"
            secret_key: "{{`AWS_SECRET_ACCESS_KEY` | env}}"
        ```

=== "**🎭 IAM Roles (AWS)**"

    **For EC2, ECS, Lambda, and other AWS services:**

    ```yaml
    # No credentials needed - uses instance/service IAM role
    storage_profiles:
      iam_role_s3:
        provider: s3
        region: us-west-2
        bucket: dbsnapper-role-based
        # Credentials automatically provided by AWS SDK
    ```

    **Required IAM Policy:**
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "s3:GetObject",
            "s3:PutObject",
            "s3:DeleteObject"
          ],
          "Resource": "arn:aws:s3:::dbsnapper-role-based/*"
        },
        {
          "Effect": "Allow",
          "Action": "s3:ListBucket",
          "Resource": "arn:aws:s3:::dbsnapper-role-based"
        }
      ]
    }
    ```

## Security Best Practices

### **📊 Implement Storage Separation**

**Use different storage profiles for different security boundaries:**

```yaml
targets:
  production_database:
    # Raw production data - highly restricted access
    storage_profile: raw_prod_s3
    
    sanitize:
      # Sanitized data - broader team access  
      storage_profile: sanitized_team_s3
      override_query: "UPDATE users SET email = CONCAT('user', id, '@example.com')"
      
    # Share only sanitized snapshots with these groups
    sso_groups: ["developers", "qa-team"]
```

### **🔒 Encryption Configuration**

**Enable encryption at rest for sensitive snapshots:**

=== "**Amazon S3**"

    ```yaml
    # Configure encryption in your S3 bucket settings
    # DBSnapper supports:
    # - SSE-S3 (Amazon S3 managed keys)
    # - SSE-KMS (AWS Key Management Service)
    # - SSE-C (Customer-provided keys)
    ```

    **S3 Bucket Policy Example:**
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "DenyInsecureConnections",
          "Effect": "Deny",
          "Principal": "*",
          "Action": "s3:*",
          "Resource": [
            "arn:aws:s3:::dbsnapper-secure/*",
            "arn:aws:s3:::dbsnapper-secure"
          ],
          "Condition": {
            "Bool": {
              "aws:SecureTransport": "false"
            }
          }
        }
      ]
    }
    ```

=== "**Cloudflare R2**"

    ```yaml
    # R2 provides encryption at rest by default
    # Configure additional security via Cloudflare dashboard:
    # - Access tokens with limited permissions
    # - API token expiration policies  
    # - IP address restrictions
    ```

### **🔍 Access Control & Monitoring**

**Implement comprehensive access controls:**

```yaml
# Production raw data - restricted access
storage_profiles:
  production_raw:
    provider: s3
    bucket: dbsnapper-production-raw
    awscli_profile: production_restricted
    # Only DevOps team has access to this profile
    
  # Team sanitized data - broader access
  team_sanitized:  
    provider: s3
    bucket: dbsnapper-team-sanitized
    awscli_profile: team_shared
    # All developers can access sanitized snapshots
```

**Monitoring and Audit:**
- Enable S3/R2 access logging to track snapshot usage
- Set up CloudTrail (AWS) or equivalent monitoring for API calls
- Configure alerts for unusual access patterns
- Regular review of storage costs and usage patterns

## Performance Optimization

### **📈 Upload/Download Performance**

**Optimize for large database snapshots:**

```yaml
# Use regions close to your database infrastructure
storage_profiles:
  optimized_s3:
    provider: s3
    region: us-west-2  # Same region as your RDS/Aurora instances
    bucket: dbsnapper-optimized
    prefix: snapshots
```

**Performance Tips:**
- **Region Selection**: Choose storage regions close to your databases
- **Multipart Uploads**: Automatically handled for large snapshots
- **Compression**: DBSnapper automatically compresses snapshots (typically 10-50% size reduction)
- **Parallel Operations**: V3.0 multi-core operations improve upload speeds

### **💰 Cost Optimization**

**Implement intelligent storage lifecycle policies:**

=== "**S3 Lifecycle Management**"

    ```json
    {
      "Rules": [
        {
          "ID": "DBSnapper-Lifecycle",
          "Status": "Enabled",
          "Filter": {
            "Prefix": "snapshots/"
          },
          "Transitions": [
            {
              "Days": 30,
              "StorageClass": "STANDARD_IA"
            },
            {
              "Days": 90,  
              "StorageClass": "GLACIER"
            }
          ],
          "Expiration": {
            "Days": 365
          }
        }
      ]
    }
    ```

=== "**R2 Cost Benefits**"

    **Cloudflare R2 advantages:**
    - Zero egress fees (major cost savings)
    - Competitive storage pricing
    - No charges for API requests
    - Global edge network for performance

## Troubleshooting

### **🔧 Common Issues & Solutions**

**Connection Test Failures:**

```bash
# Test storage profile connectivity
dbsnapper config check --storage-profile production_s3

# Test with debug information
DBSNAPPER_DEBUG=true dbsnapper config check --storage-profile production_s3
```

**Common Problems:**

=== "**AWS Credentials Issues**"

    ```
    Error: failed to list objects in bucket: NoCredentialProviders
    ```
    
    **Solutions:**
    1. Verify AWS CLI profile exists: `aws configure list --profile dbsnapper-prod`
    2. Test profile access: `aws s3 ls s3://your-bucket --profile dbsnapper-prod`
    3. Check IAM permissions include `s3:ListBucket`
    4. Ensure profile name matches storage profile configuration

=== "**Bucket Permission Issues**"

    ```
    Error: Access Denied (Service: Amazon S3; Status Code: 403)
    ```
    
    **Solutions:**
    1. Verify bucket exists and is accessible
    2. Check IAM policy includes required S3 permissions
    3. Confirm bucket region matches profile configuration
    4. Test with minimal IAM policy first, then expand

=== "**Network Connectivity Issues**"

    ```
    Error: timeout connecting to storage provider
    ```
    
    **Solutions:**
    1. Check firewall rules allow HTTPS (443) outbound
    2. Verify DNS resolution for storage endpoints
    3. Test with different regions/endpoints
    4. Check corporate proxy settings

### **🔍 Validation and Testing**

**Validate storage profile configuration:**

```bash
# Check all storage profiles
dbsnapper config validate --storage-profiles

# Test specific profile with verbose output
dbsnapper config check --storage-profile team_s3 --verbose

# Test upload/download cycle
dbsnapper build test-target --dry-run  # Verify configuration without creating snapshot
```

## Migration and Management

### **🚀 Migrating Between Storage Providers**

**Example: Migrating from S3 to R2:**

```yaml
# 1. Create new R2 storage profile
storage_profiles:
  new_r2:
    provider: r2
    bucket: migrated-snapshots
    account_id: your-r2-account
    awscli_profile: r2_profile

# 2. Update target to use new profile
targets:
  production_api:
    storage_profile: new_r2  # Changed from old S3 profile
```

**Migration Process:**
1. Create new storage profile with target provider
2. Test connectivity and permissions
3. Update targets to use new storage profile  
4. Create new snapshots with new provider
5. Optionally migrate existing snapshots using cloud provider tools

### **📋 Storage Profile Management**

**Best practices for ongoing management:**

- **Naming Convention**: Use descriptive names (`prod-raw-s3`, `team-sanitized-r2`)
- **Documentation**: Document which profiles are used by which teams
- **Regular Audits**: Review storage costs and access patterns monthly
- **Credential Rotation**: Regular rotation of access keys and secrets
- **Backup Strategy**: Consider multiple storage providers for critical snapshots

## Next Steps

**Now that you've configured storage profiles:**

1. **[Create Database Targets](targets.md)** - Define your databases and link them to storage profiles
2. **[Set up Team Access](sso/index.md)** - Configure SSO for secure team collaboration
3. **[Test Your Setup](../commands/build.md)** - Create your first cloud-stored snapshot
4. **[GitHub Actions Integration](../articles/dbsnapper-github-actions-ecs-simplified.md)** - Connect with CI/CD and development tools

---

**Need help with storage configuration?** Check our **[Configuration Settings guide](../configuration.md)** for comprehensive examples, or explore **[Cloud Storage detailed documentation](../cloud-storage-engines/introduction.md)** for advanced scenarios.

**Ready to implement complete team workflows?** Our **[Team Workflows guide](team-workflows.md)** shows how storage profiles integrate with targets, SSO, and daily development processes for enterprise-scale database snapshot collaboration.
