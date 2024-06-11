---
title: DBSnapper Cloud - Single Sign-On (SSO)
description: Configuring Single Sign-On for your DBSnapper organization
---

DBSnapper supports Single Sign-On (SSO) for organizations that want to manage user access centrally. With SSO, you can use your existing identity provider to authenticate users and manage access to DBSnapper.

We support OpenID Connect (OIDC) protocols and currently have integrations with Okta. If you have a different identity provider, contact us and we can work with you to integrate it with DBSnapper.

## Single Sign-On (SSO) benefits:

- Onboarding your team is as easy as assigning your users to the DBSnapper application in your identity provider dashboard. They can then log in to DBSnapper and their account will be automatically created.
- Snapshot Sharing: Add SSO Sharing Groups to your Target configurations to instantly share sanitized snapshots with other team members.

<p class="img-box">
  <img src="/static/cloud/targets-with-shared-targets.jpg" alt="DBSnapper Targets with Shared SSO Targets" width="90%">
  <br/>
  <small>DBSnapper Targets with Shared SSO Targets</small>
</p>

## Supported Identity Providers

Currently we support Okta as an identity provider. If you have a different identity provider, please contact us and we can add it to DBSnapper.

## Getting Started with SSO

The following guides are available to help you configure SSO with DBSnapper:

- [How to configure Okta as your SSO provider](sso-okta-oidc.md)
