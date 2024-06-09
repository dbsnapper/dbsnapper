---
title: DBSnapper Cloud Platform - Targets Page
description: The Targets page allows you to define and manage the target settings that are used to create, store, and share your database snapshots.
---

## All Targets

A **Target** is a configuration that defines the settings for creating, storing, and sharing snapshots for a single database workload. The Targets page displays all the targets you've defined along with any Shared Team Targets that you have access to. The Shared Team Targets require your organization to be using SSO as it leverages the organization's SSO groups to manage access to the shared targets.

<p class="img-box">
  <img src="/static/cloud/targets-with-shared-targets.jpg" alt="DBSnapper Cloud - All Targets" width="90%">
  <br/>
  <small>DBSnapper Cloud: Targets page including Shared Team Targets.</small>
</p>

## Create / Edit Target

To create a new target, click the `Add Target` button in the top right of the page. That will display a form similar to the Edit Targets page below where you can define the settings for the target.

<p class="img-box">
  <img src="/static/cloud/targets-edit-target.jpg" alt="DBSnapper Cloud - Edit Target" width="90%">
  <br/>
  <small>DBSnapper Cloud: Edit Targets page.</small>
</p>

On this page, you can define the following settings for the target:

### Target Details

- **Target Name**: A unique name for the target.

### Snapshot Details

- **Snapshot Database Source URL**: This is the connection string URL of the database you want to snapshot.
- **Snapshot Database Destination URL** (Optional) - This is the connection string URL of the database where you want to load the snapshot. Remember, any database specified as a destination URL will be DROPPED and RECREATED when a snapshot is loaded.
- **Snapshot Storage Profile (unsanitized)**: The storage profile (defined on the **Storage Profiles** page) where the _unsanitized_ snapshot will be stored.

### Sanitization Details

- **Sanitization Database Destination URL**: (Optional) - This is the connection string URL of the database you want to use to sanitize the snapshot. This database is used as a temporary database to sanitize the snapshot before it is dumped and stored in in the **Snapshot Storage Profile (sanitized)** location. If you leave this field empty, you can use ephemeral sanitization which will use an ephemeral Docker container to perform the sanitization operation.
- **Sharing: Share with SSO Groups** (Optional) - If your organization is enrolled in SSO, and you want to share this target's snapshots with other members of your team, you can provide a comma-delimited list of groups that should have access to the snapshots. This will allow members of the specified groups to load the **sanitized** snapshots for this target.
- **Snapshot Storage Profile (sanitized)**: The storage profile (defined on the **Storage Profiles** page) where the _sanitized_ snapshot will be stored. Specifying different storage locations for the sanitized and unsanitized snapshots allows you to control access to the sanitized snapshots.
- **Sanitization Query**: The SQL query that will be used to sanitize the snapshot. This query can be used to remove any sensitive data from the snapshot before it is stored in the **Snapshot Storage Profile (sanitized)** location.

##
