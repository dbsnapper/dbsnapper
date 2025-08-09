---
title: Sanitization Introduction
description: Introduction to the database sanitization capability of DBSnapper.
---

One of the main features of DBSnapper is the ability to sanitize a database. This allows you to remove sensitive information from your database, making it safe to use in development, testing, or analytics environments.

## Why is it Important?

- **Privacy Protection**: In today's digital world, protecting personal information is not just ethical but also a legal requirement in many cases. Sanitization helps you respect and protect user privacy.
- **Regulatory Compliance**: Laws like GDPR and HIPAA require strict handling of personal data. Sanitization ensures that you stay on the right side of these regulations.
- **Secure Development and Testing**: Using real data in development and testing can be risky if it contains sensitive information. Sanitized data allows your team to work with realistic datasets without exposing personal information.

## Overview

DBSnapper provides the [`sanitize`](../commands/sanitize.md) command to sanitize a database, which applies a sanitization query to a snapshot of the database.

### `sanitize` command steps

1. Locate the desired snapshot in the working directory or pull it from the cloud (if configured)
2. Load the snapshot into the `dst_url` database
3. Apply the sanitization query to the database
4. Create a new snapshot of the sanitized database and store it in the working directory and push it to the cloud (if configured)

!!! note

    In the near future we will be adding more capabilities to the sanitization process to make it even more powerful and flexible. Stay tuned for updates!
