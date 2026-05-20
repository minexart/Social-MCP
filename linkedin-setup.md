# LinkedIn OAuth Setup Guide

This document walks through getting your LinkedIn OAuth credentials and wiring them into the MCP server. It covers every step that's actually tricky — the ones that cost time the first time through.

---

## Prerequisites

- A LinkedIn account
- A **Company Page** you manage (not optional — see below)
- Node.js and the MCP server running locally

---

## Why a Company Page Is Required

LinkedIn's OAuth API does not allow posting to personal profiles through third-party apps. To publish posts programmatically, you must have:

1. A LinkedIn **Company Page** (also called an Organization page)
2. **Super Admin** or **Content Admin** role on that page

If you don't have a Company Page yet, create one at [linkedin.com/company/setup/new](https://www.linkedin.com/company/setup/new). It can be a personal brand, side project, or anything else — it just needs to exist.

---

## Step 1: Create a LinkedIn App

1. Go to [linkedin.com/developers/apps/new](https://www.linkedin.com/developers/apps/new)
2. Fill in the fields:
   - **App name**: anything descriptive (e.g., `Social MCP`)
   - **LinkedIn Page**: select your Company Page — **this field is required**
   - **App logo**: required; upload any image
3. Agree to the terms and click **Create app**

You'll land on your app's dashboard.

---

## Step 2: Add the Right Product

This is the step most guides skip, and it's the one that breaks everything.

1. Go to the **Products** tab of your app
2. Find **"Share on LinkedIn"** and click **Request access**
3. Wait for approval — it's usually instant or within a few minutes

> ⚠️ **Do not request "Marketing Developer Platform" (MDP).** MDP is for verified advertising partners and requires a lengthy manual approval process. The `Share on LinkedIn` product gives you the `w_member_social` scope, which is all you need for posting.

Once approved, go to the **Auth** tab. You should now see `w_member_social` available under OAuth 2.0 scopes.

---

## Step 3: Configure OAuth Settings

On the **Auth** tab:

1. Under **OAuth 2.0 Settings**, add a redirect URL:
   ```
   http://localhost:3000/callback
   ```
   (Adjust the port if your server runs on a different one.)

2. Copy your **Client ID** — you'll need this shortly.

3. Click **Generate a new Client Secret**, then copy it immediately. LinkedIn only shows it once.

> ⚠️ **Never commit the Client Secret.** Add `.env` to your `.gitignore` and keep credentials there only. In this guide, all credentials are shown as `<placeholders>`.

---

## Step 4: Configure Your `.env` File

In the project root, create or update your `.env`:

```env
LINKEDIN_CLIENT_ID=<your-client-id>
LINKEDIN_CLIENT_SECRET=<your-client-secret>
LINKEDIN_REDIRECT_URI=http://localhost:3000/callback
```

These values are loaded at runtime. Never hardcode them in source files.

---

## Step 5: Run the OAuth Flow

Start the MCP server, then visit the authorization URL in your browser:

```
https://www.linkedin.com/oauth/v2/authorization?response_type=code&client_id=<your-client-id>&redirect_uri=http://localhost:3000/callback&scope=r_liteprofile%20w_member_social
```

1. LinkedIn will ask you to log in and approve the permissions
2. After approval, LinkedIn redirects to `http://localhost:3000/callback?code=<auth-code>`
3. The server exchanges that code for an access token
4. The access token is saved (to `.env` or a local token store, depending on your implementation)

---

## Step 6: Get Your Organization URN

To post on behalf of your Company Page, you need its **Organization URN** in the format `urn:li:organization:<id>`.

Find the numeric ID in the URL when you're on your Company Page admin view:
```
https://www.linkedin.com/company/<numeric-id>/admin/
```

Then construct the URN:
```
urn:li:organization:<numeric-id>
```

Add this to your `.env`:
```env
LINKEDIN_ORGANIZATION_URN=urn:li:organization:<numeric-id>
```

---

## Troubleshooting

### `invalid_client`

**Cause:** The Client Secret in your `.env` doesn't match what LinkedIn has on record.

**Fix:** Go back to the **Auth** tab on your app, generate a new secret, and update `.env`. This is also triggered if you regenerated the secret at any point — the old one is immediately invalidated.

---

### `unauthorized_scope_error`

**Cause:** The `w_member_social` scope isn't available because you haven't been approved for the "Share on LinkedIn" product yet.

**Fix:** Go to the **Products** tab, confirm "Share on LinkedIn" shows as approved (not "pending"). If still pending, wait a few minutes and refresh.

---

### `ERR_MODULE_NOT_FOUND` or server won't start

**Cause:** The server is being run from the wrong directory, so relative paths to `node_modules` or config files don't resolve.

**Fix:** Always `cd` into the project root before running:
```bash
cd /path/to/your/project
node index.js
```

Do not run the server with an absolute path from a different working directory.

---

### The callback URL gets a `redirect_uri_mismatch` error

**Cause:** The redirect URI in your authorization request doesn't exactly match what's registered in the LinkedIn app settings — including trailing slashes and protocol (`http` vs `https`).

**Fix:** Make sure the URI in your `.env` and the URI registered in the LinkedIn developer portal are byte-for-byte identical:
```
http://localhost:3000/callback   ← registered in portal
http://localhost:3000/callback   ← in .env and in auth URL
```

---

### Posts succeed but don't appear on the Company Page

**Cause:** The post is being published to your personal profile instead of the organization.

**Fix:** Confirm that your `ugcPosts` payload uses the organization URN as the author, not the member URN:
```json
"author": "urn:li:organization:<id>"
```

---

## Security Checklist

Before pushing to GitHub or sharing the repo:

- [ ] `.env` is in `.gitignore`
- [ ] No Client Secret appears anywhere in source files or docs
- [ ] No real Client ID appears in screenshots — use placeholders or blur them
- [ ] Access tokens are stored locally only, never committed
- [ ] Demo videos have credentials blurred or replaced with fake values

---

## Reference

- [LinkedIn OAuth 2.0 documentation](https://learn.microsoft.com/en-us/linkedin/shared/authentication/authorization-code-flow)
- [Share on LinkedIn API](https://learn.microsoft.com/en-us/linkedin/consumer/integrations/self-serve/share-on-linkedin)
- [UGC Posts API](https://learn.microsoft.com/en-us/linkedin/marketing/integrations/community-management/shares/ugc-post-api)
