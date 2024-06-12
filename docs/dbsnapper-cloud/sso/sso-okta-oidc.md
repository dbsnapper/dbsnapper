---
title: DBSnapper Cloud - Single Sign-On (SSO) with Okta OIDC
description: Configure Single Sign-On for your DBSnapper organization with Okta OIDC
---

In this guide, we will use Okta as the SSO provider and with the Open ID Connect (OIDC) protocol as our identity protocol.

## Step 1: Create a new Application in Okta

Sign into your Okta account and navigate to Admin/Applications. Click on the `Create App Integration` button.

<p class="img-box">
  <img src="/static/sso/sso-okta-step1-1-create-app-integration.png" alt="Step 1.1 - Okta SSO - Applications" width="90%">
  <br/>
  <small>Step 1.1 - Okta SSO - Applications</small>
</p>

At the next modal, select `OIDC - OpenID Connect` as the Sign-in method, Choose `Web Application` as the Application type, and click `Next`.

<p class="img-box">
  <img src="/static/sso/sso-okta-step1-2-new-app-integration.png" alt="Step 1.2 - Okta SSO - New App Integration" width="90%">
  <br/>
  <small>Step 1.2 - Okta SSO - New App Integration</small>
</p>

On the next page, **New Web App Integration,** make sure `Authorization Code` and `Refresh Token` are selected.

Provide the Sign-in redirect URIs as: `https://app.dbsnapper.com/auth/okta/callback`

Leave the Sign-out redirect URIs empty.

Click `Save` to save the new application.

<p class="img-box">
  <img src="/static/sso/sso-okta-step1-3-new-web-app-integration.png" alt="Step 1.3 - Okta SSO - New Web App Integration" width="90%">
  <br/>
  <small>Step 1.3 - Okta SSO - New Web App Integration</small>
</p>

On the next scren, click on the Edit link to the right of the `Client Credentials` heading.

Take note of your application credentials which will be used in the DBSnapper Cloud SSO settings:

- Client ID - a public identifier for the client application
- Client Secret - Click to reveal the client secret and make note of it.

The rest of the settings should match what is shown in the screenshot below:

**Enable** `Proof Key for Code Exchange (PKCE)`: **Require PKCE as additional verification**

<p class="img-box">
  <img src="/static/sso/sso-okta-step1-4-edit-application.png" alt="Step 1.4 - Okta SSO - Edit Client Credentials Settings" width="90%">
  <br/>
  <small>Step 1.4 - Okta SSO - Edit Client Credentials Settings</small>
</p>

Click on the `Sign On` tab to edit the Issuer and Group claims settings.

- **Issuer** - Change it from _Dynamic_ to _Okta URL ..._ and take note of this value
- **Group claims** - Change it to match the screenshot below. `groups Matches regex .*` This will allow DBSnapper to use the groups claim for sharing snapshots with your team.

<p class="img-box">
  <img src="/static/sso/sso-okta-step1-5-sign-on-settings.png" alt="Step 1.5 - Okta SSO - Edit Sign On Settings" width="90%">
  <br/>
  <small>Step 1.5 - Okta SSO - Edit Sign On Settings</small>
</p>

## Step 2: Configure DBSnapper for Okta OIDC

Sign into your DBSnapper Cloud account and navigate to the `Settings -> SSO` page. Click on the `No SSO Tenant Found - Add a Tenant` button.

Add the settings from the Okta application you created in Step 1:

- **Provider** - Okta - OIDC (the default)
- **Client ID** - from the `General` tab of the Okta application
- **Client Secret** - from the `General` tab of the Okta application
- **Issuer** - from the `Sign On` tab of the Okta application

Note that the **Organization Domain** and **Redirect URL** are pre-filled for your organization.

<p class="img-box">
  <img src="/static/sso/sso-okta-step2-1-dbsnapper-new-sso-tenant.png" alt="Step 2.1 - Okta SSO - Create new SSO Tenant in DBSnapper" width="90%">
  <br/>
  <small>Step 2.1 - Okta SSO - Create new SSO Tenant in DBSnapper</small>
</p>

Click `Save` to save the new SSO Tenant for your organization. You should see your new SSO Tenant settings shown on the SSO page.

<p class="img-box">
  <img src="/static/sso/sso-okta-step2-2-dbsnapper-sso-tenant-created.png" alt="Step 2.2 - Okta SSO - SSO Tenant Created in DBSnapper" width="90%">
  <br/>
  <small>Step 2.2 - Okta SSO - SSO Tenant Created in DBSnapper</small>
</p>

Congratulations you have successfully configured Okta OIDC for your DBSnapper organization.
