# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Qwello Site Import is a zero-dependency, no-build single-page application for Qwello employees to bulk-import EV charging sites into **SiteTracker** (a Salesforce-based system). It is hosted on GitHub Pages at `https://wolfster.github.io/qwello-site-import/` and all app logic lives in a single file: `index.html`.

## Development Commands

Start a local dev server (no build step required):

```bash
# Option 1 — Python (no install needed)
python3 -m http.server 8080

# Option 2 — npx serve
npx serve -p 8080
```

Open `http://localhost:8080` in Chrome. Use **Demo Mode** (button on login screen) to work without a Salesforce connection.

There are no tests, no linters, and no build pipeline.

## Architecture

### Single-File SPA (`index.html`, ~1,800 lines)

The entire application — HTML, CSS, and JavaScript — is in `index.html`. The file is organized in three clear sections:

1. **CSS** (lines ~9–263): CSS custom properties for the dark theme (`--bg`, `--surface`, `--accent`, `--error`), CSS Grid/Flexbox layouts, mobile breakpoint at 600px.
2. **HTML** (lines ~268–684): Login screen, 5-screen wizard shell, confirm modal, toast notifications, loading overlay.
3. **JavaScript** (line ~686 onward): Single global state object `S`, all functions, no modules.

### Global State (`S`)

All runtime state lives in the `S` object:
- `S.token` — Salesforce session token (from `sessionStorage`)
- `S.rows` — parsed site rows from pasted Excel data
- `S.mapping` — column-to-field assignments from step 3
- `S.importResults` — per-row success/error results from step 5
- `S.programs`, `S.users` — Salesforce lookup cache

### 5-Step Wizard

| Step | Screen ID | Purpose |
|------|-----------|---------|
| 1 | `screen-global` | Country, Unicode ID, Program, Sales Manager, Project Manager |
| 2 | `screen-paste` | Paste tab-separated Excel data |
| 3 | `screen-map` | Map pasted columns → Salesforce API field names |
| 4 | `screen-review` | Inline cell editing before import |
| 5 | `screen-results` | Per-row success/error stats, CSV download |

Screen transitions are managed by `showScreen(id)`. Validation for each step runs in `validateStep(n)` before advancing.

### Salesforce Integration

**Object:** `sitetracker__Site__c`  
**API version:** `v59.0`  
**Base domain:** `https://sitetracker-qwello.my.salesforce.com`

Key constants at the top of the script block:
```js
const SF_DOMAIN   = 'https://sitetracker-qwello.my.salesforce.com';
const CLIENT_ID   = '3MVG92fMZeJeHJdodQ_Lga8Dk2NnI9BeHVD2dMF7By6...';
const REDIRECT_URI = 'https://wolfster.github.io/qwello-site-import/callback.html';
const SITE_OBJECT = 'sitetracker__Site__c';
```

**Authentication (current):** Users paste a Salesforce session token (`sid` cookie value from Chrome DevTools). Token is stored only in `sessionStorage`.

**Authentication (planned, blocked):** OAuth 2.0 PKCE via `callback.html`. The org uses an External Client App (not a classic Connected App), which does not support browser-based OAuth from external domains. A classic Connected App with "Authorization Code + PKCE" is needed but requires "Customize Application" permission that the current user lacks.

**Live lookups** (`filterPrograms()`, `updateUsersByCountry()`): 2-character minimum, 350 ms debounce, results cached in `S.programs`/`S.users`.

**Bulk insert:** Salesforce Collections API (`POST /composite/sobjects`), batched at 200 records per call.

### UNLOCODE Data

Seven country files (`unlocode-de.js`, `unlocode-fr.js`, etc.) each export an array of `[code, name]` tuples. Files are lazy-loaded (`<script>` tag appended to `<head>`) when the user selects a country, then cached so they load only once per session. Autocomplete filters by city name substring; if no match, the user can enter a custom 3-letter code manually.

### Picklist Values

All Salesforce picklist values are **hardcoded** in the `PICKLIST_DATA` object (captured April 2025). They are not fetched live. Fields with hardcoded picklists:
- `sitetracker__Site_Status__c` (13 values)
- `Site_Status_2026__c` (6 values — controls visibility of Rejection fields)
- `Country__c` (7 values)
- `Pole_Model__c`, `Location_Design__c`, `Location_Setup__c`
- `Rejection_Actor__c`, `Rejection_Reason__c`

### Demo Mode

Activated by "Try Demo Mode" on the login screen. Replaces all Salesforce API calls with local mock data (3 programs, 4 users, 10 pre-filled Berlin/München/Hamburg sites). Import simulation returns ~85% success. Demo mode is the correct way to test UI changes without Salesforce credentials.

## Key Conventions

- **No frameworks, no bundler.** Keep it that way — adding npm dependencies would break the GitHub Pages zero-build deploy.
- **State mutations go through `S`.** Do not introduce module-level variables for runtime state.
- **Field API names** (e.g., `Unicode_ID__c`, `sitetracker__Lat__c`) are used as keys throughout the mapping, validation, and export logic. Match Salesforce exactly.
- **Rejection section** is conditionally rendered based on `Site_Status_2026__c === 'Rejected'`. This conditional appears in both the review table renderer and the CSV export.
- **CSV export** uses `YYYY-MM-DD` for dates, omits empty optional fields, and writes Salesforce IDs (not labels) for lookup fields.
- **Security:** Keep tokens in `sessionStorage` only. Do not add `localStorage` persistence for credentials. The CSP in the `<meta>` tag must be updated if new external domains are fetched.

## Important Docs in This Repo

- `DEVELOPER_PROMPT.md` — Complete field list, auth flow, API patterns (primary technical reference)
- `PROJEKT_STATUS.md` — Current blockers and known issues (read before starting new features)
- `SALESFORCE_DEV_HANDOFF.md` — OAuth problem summary for the Salesforce developer
- `ANLEITUNG_MITARBEITER.md` — End-user guide in German
