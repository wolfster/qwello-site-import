# Salesforce Developer Handoff — Qwello Site Import Tool

## Context

We are building an internal web tool for Qwello to bulk-import charging sites into SiteTracker.
The tool is a pure HTML/JS single-page application hosted on **GitHub Pages** (no backend server).

**Live URL:** https://wolfster.github.io/qwello-site-import/
**Salesforce Org:** https://sitetracker-qwello.my.salesforce.com

The tool needs to authenticate users against Salesforce via OAuth, then use the Salesforce REST API
to query data (programs, users) and insert records via the Collections API.

---

## The Problem

We cannot get OAuth to work. The org currently has an **External Client App** (not a classic
Connected App), and we have tried every available OAuth flow — none of them work from a
browser-based app hosted on an external domain.

---

## What We Have Tried

### Attempt 1 — Standard PKCE Authorization Code Flow
- Endpoint: `GET /services/oauth2/authorize`
- Parameters: `response_type=code`, `client_id`, `redirect_uri`, `scope=api id`,
  `code_challenge`, `code_challenge_method=S256`
- **Result:** `{"error":"invalid_client_id","error_description":"client identifier invalid"}`
- **Reason:** This endpoint does not recognize External Client App consumer keys.

### Attempt 2 — External Client App Authorization Endpoint
- Endpoint: `GET /services/auth/oauth2/authorize`
- Same parameters as above.
- **Result:** Error page with `Bad_Id`
- **Note:** Tested after changing Distribution State from "Local" to "Packaged". Same error.

### Attempt 3 — Device Flow
- Enabled "Enable Device Flow" in the External Client App settings.
- POST to `/services/oauth2/token` with `grant_type=device&client_id=...`
- **Result:** `{"error":"invalid_request","error_description":"invalid device code"}`
- **Reason:** External Client Apps do not support Device Flow from external browser origins.

### Configuration Already Done
- `https://wolfster.github.io` is whitelisted in Salesforce CORS settings ✓
- "Enable CORS for OAuth endpoints" checkbox is enabled ✓
- Flow enabled: Authorization Code and Credentials Flow ✓
- PKCE required: checked ✓
- Distribution State: Packaged ✓

---

## What We Need

A classic Salesforce **Connected App** configured as follows:

| Setting | Value |
|---------|-------|
| OAuth Flow | Authorization Code and Credentials (PKCE required) |
| Callback URL | `https://wolfster.github.io/qwello-site-import/callback.html` |
| OAuth Scopes | `Access and manage your data (api)`, `Access your basic information (id)` |
| CORS | Allow from `https://wolfster.github.io` |
| Require Secret for Web Server Flow | No (PKCE only, no client secret) |

Once the Connected App is created, please share:
1. The **Consumer Key** (Client ID)
2. Confirmation that the callback URL is set correctly

We will update two constants in our code:
- `CLIENT_ID` in `index.html` (~line 30)
- `CLIENT_ID` in `callback.html` (~line 39)

---

## Current Consumer Key (External Client App — not working)

```
3MVG92fMZeJeHJdodQ_Lga8Dk2NnI9BeHVD2dMF7By6.4pSxBcLtlpFtSla3sFs7XS4rAwltO3VZW8tCsGn3
```

---

## Why We Can't Create It Ourselves

The Salesforce user account does not have the **"Customize Application"** permission,
which is required to create Connected Apps in Setup → App Manager.

---

## Technical Notes

- The callback page (`callback.html`) exchanges the authorization code for a token using PKCE
  and stores `access_token` and `instance_url` in `sessionStorage`. This code is already
  fully implemented and tested — it only needs a valid Connected App consumer key to work.
- No client secret is used (pure PKCE flow, safe for browser-based apps).
- The tool only needs read access to `Program__c` and `User` objects, and write access to
  `sitetracker__Site__c` (the SiteTracker site object).
