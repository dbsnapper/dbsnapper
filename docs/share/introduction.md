---
title: Sharing Snapshots
description: Sharing database snapshots with DBSnapper.
---

# Introduction

One of the key DBSnapper features is the ability to easily share sanitized database snapshots with your team, developers, testers, and other stakeholders. Sharing snapshots is useful for providing sanitized data for development, testing, and other purposes without exposing sensitive information.

By utilizing your own private cloud storage providers, you can securely store your snapshots in your own approved infrastructure and use the DBSnapper Agent and Cloud to facilitate easy sharing.

## Sharing Architecture

![DBSnapper Sharing Architecture](/static/vpc-s3-horiz.svg)

## Getting Started

To get started with using a cloud storage engine, you will need to configure a `storage_profile` in your DBSnapper configuration file. The `storage_profile` configuration specifies the cloud storage provider, credentials, and bucket information for the storage engine.

<!-- prettier-ignore-start -->
!!! example "AWS S3 configuration examples"
  
    ```yaml linenums="1" hl_lines="11-15 22"
    storage_profiles:
      s3-with-provided-credentials:
        provider: s3
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        region: <region>
        bucket: dbsnapper-test-s3
        prefix:

      s3-from-awscli-shared-profile:
        provider: s3
        awscli_profile: dbsnapper_credentials
        bucket: dbsnapper-test-s3
        prefix:

    targets:
      # Share target
      shared-s3:
        name: shared-s3
        share:
          storage_profile: s3-from-awscli-shared-profile
          dst_url: postgres://localhost:5432/dbsnapper_test
    ```
<!-- prettier-ignore-end -->

In the example above we have two `storage_profiles` configurations for AWS S3. The first configuration `s3-with-provided-credentials` explicitly specifies the access key, secret key, region, and bucket for an S3 storage engine. `s3-from-awscli-shared-profile`, on the other hand, indicates that we want to retrieve the credentials from the `dbsnapper_credentials` AWS shared configuration profile as specified in the `awscli_profile` field.

We have also defined a `share target` configuration on line 19 that uses the `s3-from-awscli-shared-profile` storage profile to access shared snapshots from the `dbsnapper-test-s3` bucket. These shared snapshots will be loaded into the `dbsnapper_test` database as specified in the `dst_url` field on line 23.
