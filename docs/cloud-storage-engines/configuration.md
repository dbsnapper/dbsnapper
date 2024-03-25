---
title: Storage Engine Configuration
description: Options for configuring cloud storage engines in DBSnapper.
---

# Storage Engine Configuration

You specify configurations for storage engines in the DBSnapper configuration file under the `storage_profiles` section. These storage engines can be used with sharing targets to retrieve shared snapshots from cloud storage.

Several variations of `storage_profile` configurations are shown in the following example for different cloud storage providers and methods of retrieving credentials.

## Example

<!-- prettier-ignore-start -->
!!! example "`storage profiles` configuration"
  
    ```yaml
    # Storage profiles for sharing configurations
    storage_profiles:
      # All Profiles:
      # Leave access_key and secret_key to use a profile
      # Leave the awscli_profile empty to use the default profile
      # If the endpoint is specified in the profile it will override the
      # endpoint specified in this configuration.

      # S3
      s3-from-credentials:
        provider: s3
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        region: <region>
        bucket: dbsnapper-test-s3
        prefix:

      s3-from-awscli-profile:
        provider: s3
        awscli_profile: dbsnapper_credentials
        bucket: dbsnapper-test-s3
        prefix:

      # R2
      # Uses the account ID to create the endpoint url.
      # Can be omitted if the r2 endpoint_url is specified in the awscli config.
      # Or set the following Env Variables and leave the awscli_profile empty:
      # AWS_ACCESS_KEY
      # AWS_SECRET_KEY"
      # AWS_ENDPOINT_URL"
      r2-from-credentials:
        provider: r2
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        account_id: <account_id>
        bucket: dbsnapper-test-r2
        prefix:

      r2-from-awscli-profile:
        provider: r2
        awscli_profile: r2_production
        account_id: # optional if enpoint_url set in profile
        bucket: dbsnapper-test-r2
        prefix: 

      r2-from-env-or-default-awscli_profile:
        provider: r2
        bucket: dbsnapper-test-r2
        prefix: sanitized

      # Minio
      minio-from-credentials:
        provider: minio
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        endpoint: http://localhost:9000
        bucket: dbsnapper-test-minio
        prefix:

      # Digital Ocean Spaces
      # Endpoint shold be set to the endpoint of the spaces region i.e. nyc3 below
      dospaces-from-credentials:
        provider: dospaces
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        endpoint: https://nyc3.digitaloceanspaces.com
        bucket: dbsnapper-test-do
        prefix:
    ```
<!-- prettier-ignore-end -->

## Configuration Options

Storage profile configuration options essentially consist of a section to configure credentials and service endpoints, and another section for specifying the `bucket` and optional `prefix` for snapshots.

<!-- prettier-ignore-start -->
!!! info "Storage Profile configuration options (* denotes required)"

    `provider` *
    
    :   The name of the cloud storage provider. Supported providers are `s3`, `r2`, `minio`, and `dospaces`

    `awscli_profile`

    :   If you have configured the AWS CLI with named profiles, you can specify the profile name here. If this is not specified and the `access_key` and `secret_key` are omitted, environment variables or the default profile will be used.

    `access_key`

    :   The access key for the cloud storage provider. This can be omitted if the `awscli_profile` is specified or you have the appropriate environment variables set

    `secret_key`

    :   The secret key for the cloud storage provider. This can be omitted if the `awscli_profile` is specified or you have the appropriate environment variables set

    `region`

    :   The region for the cloud storage provider. This is required for AWS S3 storage profiles.

    `account_id`

    :   The account ID for the Cloudflare R2 storage provider. This is required for R2 storage profiles and is used in creating the endpoint URL.

    `endpoint`

    :   The endpoint URL for the cloud storage provider. This is required for Minio and Digital Ocean Spaces storage profiles, and is automatically generated for R2 storage profiles using the account ID.

    `bucket` *

    :   The name of the bucket in the cloud storage provider where snapshots will be stored.

    `prefix`

    :   An optional prefix to use for the snapshots in the cloud storage provider. This can be used to organize snapshots in the bucket.

<!-- prettier-ignore-end -->

## AWS Shared Configuration

DBSnapper supports retrieving credentials from the <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html" target="_blank">AWS CLI shared configuration.</a>

### Credentials Retrieval

Credentials are retrieved from the shared configuration in the following order:

1. If the `access_key` and `secret_key` are specified in the storage profile configuration, those credentials will be used and the shared configuration will be ignored.
2. The envirnoment variables `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` will be used if they are set. Along with `AWS_REGION` and `AWS_ENDPOINT_URL` if they are needed for the storage provider.
3. If the `awscli_profile` is specified in the storage profile configuration, the credentials for the specified profile will be used.
4. If the `awscli_profile` is not specified and the above values are not present, the default profile (if it exists) will be used.

### Environment Variables

If the `awscli_profile` is not specified and the `access_key` and `secret_key` are not set, the following environment variables can be used to set the credentials:

- `AWS_ACCESS_KEY`
- `AWS_SECRET`
- `AWS_REGION`
- `AWS_ENDPOINT_URL`

See the <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html" target="_blank">AWS CLI shared configuration</a> documentation for more information on supported environment variables.

### Endpoint URL

Non AWS S3 storage providers require the `endpoint` to be specified in the storage profile configuration. This is the URL for the storage provider's API. For R2 storage profiles, the endpoint URL is automatically generated using the account ID.

You may also provide the <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-endpoints.html" target="_blank">endpoint URL in the AWS CLI shared configuration</a>. If the endpoint is specified in the profile, it will override the endpoint specified in the configuration.
