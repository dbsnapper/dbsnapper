# How DBSnapper Works

## Step 1: Snapshot Your Database

<div style="text-align: center;">
  <img src="/static/hiw/hiw-snapshot-2x.png" style="width: 120px; float: left; margin-right: 10px;">
</div>

Begin by using the DBSnapper Command Line Interface (CLI) to create a snapshot of your database. This snapshot captures a point-in-time backup of the current state of your database. Use the DBSnapper cloud to upload the snapshots into your own private storage provider

---

## Step 2: Sanitize Sensitive Data

<div style="text-align: center;">
  <img src="/static/hiw/hiw-sanitize-2x.png" style="width: 120px; float: right; margin-left: 10px;">
</div>

The next crucial step is sanitizing your snapshot. DBSnapper provides tools to remove sensitive data and Personally Identifiable Information (PII) efficiently. This process is essential for maintaining privacy and compliance with data protection regulations. The de-identification feature ensures that your development and testing environments are safe for use without compromising on data integrity.

DBSnapper allows you to remove sensitive data and Personally Identifiable Information (PII) from the snapshot, ensuring de-identification is straightforward and efficient.

---

## Step 3: Share with your team

<div style="text-align: center;">
  <img src="/static/hiw/hiw-share-2x.png" style="width: 150px; float: left; margin-right: 10px;">
</div>

After sanitization, it's time to share the snapshot with your team. DBSnapper facilitates easy and secure sharing directly from your private cloud storage. This step is instrumental in speeding up the software development process, as your team gets access to realistic, yet secure, user data. Whether it’s for development, testing, or staging, your team can work with data that mirrors the production environment, enhancing the accuracy and efficiency of your workflows.

---

## Get Started

Ready to get started with DBSnapper? Head over to the [Installation](installation.md) guide to install DBSnapper on your system. Once installed, you can follow the [Quick Start](quick-start.md) guide to create your first snapshot.
