# Pull Command

The `pull` command downloads snapshots from cloud storage to your local working directory. This essential cloud integration command enables access to snapshots stored in S3-compatible storage, facilitating team collaboration and cross-environment snapshot sharing.

## Overview

The pull command bridges cloud and local storage by:
- Downloading snapshots from configured cloud storage profiles
- Automatically handling sanitized vs original snapshot selection
- Providing fast access to team-shared snapshots
- Supporting both cloud targets and shared targets from Okta SSO
- Maintaining local cache for improved performance

## Syntax

```bash
dbsnapper pull <target_name> [snapshot_index] [flags]
```

## Arguments

- `<target_name>`: The name of the target defined in your configuration file
- `[snapshot_index]`: Optional index of the snapshot to pull (default: 0 for most recent)

## Options

### Command-Specific Flags

```bash
  -o, --original   Use the original snapshot instead of the sanitized version
  -h, --help       help for pull
```

### Global Flags

```bash
      --config string   Config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

## Target Configuration

The pull command requires targets with cloud storage configuration:

### Required Configuration

**Target with Storage Profile**:

```yaml
storage_profiles:
  aws-s3-prod:
    type: s3
    region: us-west-2
    bucket: company-snapshots
    access_key_id: ${AWS_ACCESS_KEY_ID}
    secret_access_key: ${AWS_SECRET_ACCESS_KEY}

targets:
  production-db:
    name: "Production Database"
    storage_profile: "aws-s3-prod"  # Links to storage profile
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/app"
      dst_url: "postgresql://user:pass@dev:5432/app_copy"
```

### Authentication Requirements

- **Cloud Targets**: Requires DBSnapper Cloud API token
- **Storage Access**: Requires valid storage credentials (AWS keys, etc.)
- **Shared Targets**: Requires Okta SSO authentication

## Example Usage

### Basic Pull Operations

```bash
# Pull the most recent snapshot (sanitized priority)
dbsnapper pull production-db

# Pull specific snapshot by index
dbsnapper pull production-db 3

# Pull original snapshot instead of sanitized
dbsnapper pull production-db --original
dbsnapper pull production-db 2 --original
```

### Team Collaboration Examples

```bash
# Pull shared team snapshot
dbsnapper pull team-database

# Pull from different environments
dbsnapper pull staging-db 0        # Latest staging snapshot
dbsnapper pull production-db 5     # Specific production snapshot
```

## How It Works

1. **Target Validation**: Verifies the target exists and has cloud storage configured
2. **Snapshot Selection**: Identifies the requested snapshot by index (0 = most recent)
3. **Type Resolution**: Determines whether to pull original or sanitized version
4. **Cloud Download**: Downloads the snapshot from S3-compatible storage
5. **Local Storage**: Saves the snapshot to your working directory
6. **Verification**: Confirms successful download and file integrity

## Sanitized vs Original Snapshots

### Default Behavior (Sanitized Priority)

```bash
dbsnapper pull production-db 0
```
- **Prioritizes sanitized snapshots** for data protection
- Downloads `.san.zip` version if available
- Falls back to original if no sanitized version exists
- **Use case**: Safe data sharing, development environments

### Original Snapshot Access

```bash
dbsnapper pull production-db --original
```
- **Forces original snapshot download** (`.zip` version)
- Bypasses sanitized version even if available
- **Use case**: Full data access, production debugging

### Snapshot Type Resolution

| Sanitized Available | Original Available | Default Behavior | With `--original` |
|---------------------|-------------------|------------------|-------------------|
| ✅ Yes | ✅ Yes | Downloads sanitized | Downloads original |
| ✅ Yes | ❌ No | Downloads sanitized | Downloads sanitized |
| ❌ No | ✅ Yes | Downloads original | Downloads original |
| ❌ No | ❌ No | Error: snapshot not found | Error: snapshot not found |

## Cloud Storage Integration

### Supported Storage Types

- **Amazon S3**: Native S3 API support
- **S3-Compatible**: MinIO, Cloudflare R2, DigitalOcean Spaces
- **Future Support**: Azure Blob, Google Cloud Storage

### Storage Profile Configuration

```yaml
storage_profiles:
  # Amazon S3
  aws-production:
    type: s3
    region: us-east-1
    bucket: prod-snapshots
    access_key_id: ${AWS_ACCESS_KEY_ID}
    secret_access_key: ${AWS_SECRET_ACCESS_KEY}
  
  # Cloudflare R2
  cloudflare-r2:
    type: s3
    region: auto
    bucket: team-snapshots
    endpoint_url: https://account-id.r2.cloudflarestorage.com
    access_key_id: ${CF_R2_ACCESS_KEY_ID}
    secret_access_key: ${CF_R2_SECRET_ACCESS_KEY}
```

### Authentication Setup

**Environment Variables (Recommended)**:
```bash
export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"
export DBSNAPPER_API_TOKEN="your_cloud_token"
```

**Configuration File**:
```yaml
auth:
  cloud_token: ${DBSNAPPER_API_TOKEN}
```

## Target Types

### Cloud Targets

Targets managed through DBSnapper Cloud with full API integration:

```yaml
targets:
  cloud-managed:
    name: "Cloud Managed Target"
    storage_profile: "aws-s3-prod"
    # Additional configuration managed through cloud interface
```

### Shared Targets

Targets shared via Okta SSO integration for team access:

```yaml
targets:
  shared-team-db:
    name: "Shared Team Database" 
    storage_profile: "team-storage"
    # Requires Okta SSO authentication
```

## Use Cases

### Development Environment Sync

```bash
# Keep development in sync with latest production snapshot
dbsnapper pull production-db
dbsnapper load dev-environment
```

### Cross-Team Collaboration

```bash
# Access snapshots created by other team members
dbsnapper pull shared-analytics-db
dbsnapper load local-analytics
```

### Environment Promotion

```bash
# Pull specific tested snapshot from staging
dbsnapper pull staging-db 5
dbsnapper load production-db
```

### Historical Data Access

```bash
# Access older snapshots for analysis
dbsnapper pull user-db 10    # 10 snapshots ago
dbsnapper load analysis-env
```

## Troubleshooting

### Common Issues

**Target Not Found**
```
Error: target 'production-db' not found
```
- Verify target exists in configuration
- Check target spelling and case sensitivity
- Ensure target has storage_profile configured

**Storage Authentication Failed**
```
Error: failed to authenticate with cloud storage
```
- Verify storage profile credentials
- Check environment variables are set
- Confirm access keys have appropriate permissions

**Snapshot Not Found**
```
Error: snapshot index 3 not found for target production-db
```
- List available snapshots with `dbsnapper target production-db`
- Use valid snapshot index (0 = most recent)
- Check if snapshots exist in cloud storage

**Network Connection Issues**
```
Error: failed to download snapshot from cloud storage
```
- Check internet connectivity
- Verify cloud storage endpoint accessibility
- Confirm firewall/proxy settings allow cloud access

**Insufficient Permissions**
```
Error: access denied to cloud storage bucket
```
- Verify storage credentials have read permissions
- Check bucket-level policies and access controls
- Ensure API token has appropriate cloud access

### Recovery Steps

1. **Verify Configuration**: Check target and storage profile setup
2. **Test Authentication**: Confirm cloud credentials are valid
3. **Check Connectivity**: Ensure cloud storage is accessible
4. **List Snapshots**: Use `dbsnapper target <name>` to see available snapshots
5. **Try Different Index**: Access different snapshots if specific index fails

## Performance Optimization

### Local Caching

- Downloaded snapshots are cached locally
- Subsequent pulls check local cache first
- Use `--force` flag to bypass cache (when available)

### Parallel Downloads

- Large snapshots downloaded in parallel chunks
- Configurable chunk size for optimal performance
- Automatic retry on failed chunks

### Bandwidth Management

- Progress indicators for large downloads
- Resumable downloads for interrupted transfers
- Compression optimization during transfer

!!! note "Sanitized Priority"
    By default, pull prioritizes sanitized snapshots (`.san.zip`) over original snapshots (`.zip`) for data protection. Use `--original` to override this behavior.

!!! tip "Team Collaboration"
    Use shared targets with Okta SSO integration for seamless team access to snapshots without managing individual credentials.

## Related Commands

- [`build`](build.md) - Create snapshots for cloud storage
- [`load`](load.md) - Load pulled snapshots into databases
- [`target`](target.md) - View available snapshots for a target
- [`targets`](targets.md) - List all configured targets

## See Also

- [Cloud Storage Setup](../cloud-storage-engines/introduction.md) - Configure storage profiles
- [Team Collaboration](../dbsnapper-cloud/targets.md) - Shared targets and SSO
- [Configuration](../configuration.md) - Target and storage configuration