---
title: DBSnapper Cloud Platform - Storage Profiles Page
description: The Storage Profiles page allows you to define and manage the storage profiles that are used to store your database snapshots.
---

## All Storage Profiles

A **Storage Profile** specifies the access credentials and storage bucket / prefix for storing a snapshot in a cloud storage provider.

<p align="center">
  <img src="/static/cloud/storage-profiles-index.png" alt="DBSnapper Cloud - All Storage Profiles" width="90%">
  <br/>
  <small>DBSnapper Cloud Storage Profiles page.</small>
</p>

The Storage Profiles page displays all the storage profiles you've defined. These storage profiles are used in the create/edit Targets page, where you can specify different storage profiles for the unsanitized and sanitized snapshots.

## Create / Edit Storage Profile

To create a new Storage Profile, click the `New Storage Profile` button in the top right of the page. That will display a form similar to the Edit Storage Profile page below where you can define the settings for the Storage Profile.

<p align="center">
  <img src="/static/cloud/storage-profiles-edit.png" alt="DBSnapper Cloud - Edit Storage Profile" width="90%">
  <br/>
  <small>DBSnapper Edit Storage Profile page.</small>
</p>

On this page, we are editing the storage profile settings for an Amazon S3 bucket. The following settings are available:

### Profile Name

- **Name**: A unique name for the storage profile.

### Storage Provider Info

- **Provider**: This is a dropdown that provides a list of supported cloud storage providers. Currently, the supported providers are Amazon S3 and Cloudflare R2.
- **Region / Account ID**: Depending on the provider you select, you will need to provide the region (AWS S3) or your account ID (Cloudflare R2) for the storage provider.
- **Access Key ID**: The access key ID for the storage provider IAM account.
- **Secret Access Key**: The secret access key for the storage provider IAM account.

### Storage Location

- **Bucket** the name of the bucket where the snapshot will be stored.
- **Prefix** (Optional) - A prefix that will be added to the snapshot name when it is stored in the bucket. This can be useful for organizing snapshots in the bucket.
