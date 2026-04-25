# Qwello Site Import Tool — Projektstatus

## Was ist das Tool?

Ein internes Web-Tool für Qwello zum Bulk-Import von Standorten (Sites) in SiteTracker (Salesforce).
Reines HTML/CSS/JS — kein Backend, kein Build-Step, gehostet auf GitHub Pages.

**Live-URL:** https://wolfster.github.io/qwello-site-import/
**Repo:** https://github.com/wolfster/qwello-site-import

---

## Was wurde gebaut

### Kernfunktion (4-Schritt-Wizard)

1. **Global Settings** — Batch-Name, Land, Unicode ID, Programm, Sales Manager, Projektmanager
2. **Paste Data** — Excel-Tabelle per Copy-Paste einfügen (Tab-separiert)
3. **Map Columns** — CSV-Spalten den SiteTracker-Feldern zuordnen (Auto-Mapping + manuell)
4. **Review & Import** — Daten prüfen, Zellen editieren, Import in SiteTracker auslösen
5. **Results** — Erfolge und Fehler pro Zeile, Fehler-CSV Download

### Demo Mode
- Vollständig funktionsfähiger Demo-Modus ohne Salesforce-Verbindung
- Startet über "Try Demo Mode" auf dem Login-Screen
- 10 vorausgefüllte Beispiel-Standorte (Berlin, München, Hamburg, Köln, Frankfurt)
- Demo-Programme: Berlin 2025, München 2025, Hamburg 2025
- Demo-User: Sales 1, Sales 2, PM 1, PM 2
- Simulierter Import mit ~85% Erfolgsrate und einem Demo-Fehler
- "⚡ Load Demo Data" Button in Step 2

### UNLOCODE Autocomplete
- Stadtname eingeben → UNLOCODE-Vorschläge → Unicode ID wird automatisch befüllt
- Unterstützte Länder: Deutschland (10.009), Frankreich (14.378), Spanien (4.765),
  UK (5.866), Polen (1.949), Schweden (1.198), Dänemark (753)
- Datensätze lazy-geladen (nur wenn Land ausgewählt), danach gecacht
- Quelle: UNECE Official Release 2024-2
- Wenn kein Eintrag gefunden: eigenen 3-Buchstaben-Code eingeben (Custom Code)

### Salesforce OAuth (PKCE Flow)
- Callback-Handler in `callback.html` vollständig implementiert
- `startLogin()` baut PKCE-Challenge und leitet zu Salesforce weiter
- Token wird in `sessionStorage` gespeichert (sf_token, sf_instance, sf_user)
- **→ Aktuell blockiert (siehe unten)**

### Technische Details
- Collections API (`/composite/sobjects`) für Bulk-Insert
- Autocomplete-Lookups für Program, Sales Manager, Project Manager
- Länder-Dropdown aus Salesforce (`Country__c` Picklist)
- Fallback-Länderliste wenn Salesforce nicht erreichbar
- Alle Felder aus FIELDS-Array in `index.html` konfigurierbar

---

## Security Hardening (April 2025)

Durchgeführt nach einem internen Security-Audit (zwei unabhängige Experten-Reviews).

### Implementierte Fixes

| # | Was | Datei |
|---|-----|-------|
| K1 | OAuth CSRF: State-Parameter Validierung in Callback (forward-compatible) | `callback.html` |
| K2 | Token-Leak-Schutz: `data.id`-URL wird gegen `*.salesforce.com` validiert | `callback.html`, `index.html` |
| K3 | XSS-Fix: UNLOCODE-Autocomplete nutzt jetzt `data-code`/`data-name`-Attribute statt unsicherer `onclick`-Strings; alle Namen via `escHtml()` escaped | `index.html` |
| H3 | `sessionStorage.clear()` wird sofort bei 401-Fehler aufgerufen (nicht erst beim Logout) | `index.html` |
| M1 | `ipapi.co` (Drittanbieter-IP-Geolokation) ersetzt durch browser-native Timezone-Erkennung (`Intl.DateTimeFormat`) — kein externer Call mehr | `index.html` |
| M2 | Persönliche E-Mail-Adresse aus öffentlicher Dokumentation entfernt | `DEVELOPER_PROMPT.md` |
| M4 | CSV Formula Injection Schutz: Felder die mit `=`, `+`, `-`, `@` beginnen werden mit `'` präfixiert | `index.html` |
| L1 | Clickjacking-Schutz: JS frame-buster + `frame-ancestors 'none'` in CSP | `index.html` |
| L3 | Referrer Policy: `no-referrer` → `strict-origin-when-cross-origin` | `index.html` |

### Noch offen (extern / bewusst zurückgestellt)

| # | Was | Warum offen |
|---|-----|-------------|
| H1 | Salesforce CORS-Whitelist prüfen: nur `wolfster.github.io` erlaubt? | Salesforce Admin erforderlich |
| H1 | Redirect-URI in Connected App auf bekannte Callback-URL beschränken | Salesforce Admin erforderlich |
| M3 | Demo-Daten anonymisieren (Städtenamen entfernen) | Niedrige Priorität |
| ~~L4~~ | ~~SRI-Integritätsprüfung für UNLOCODE-Dateien~~ | Nicht relevant — Daten sind öffentlich (UNECE), kein Schutzbedarf. Code kommt im Normalfall aus SiteTracker. |

### Akzeptierte Risiken (nicht behebbar ohne Backend)

- `unsafe-inline` in CSP — unvermeidbar bei Single-File-Architektur
- Session-Token in `sessionStorage` — Best-Practice für GitHub Pages ohne Backend
- CLIENT_ID im öffentlichen Repo — akzeptables Risiko bei PKCE ohne Client Secret

---

## Bekannte Probleme / Blocker

### 🔴 KRITISCH: Salesforce OAuth funktioniert nicht

**Problem:** Das Tool verwendet eine Salesforce *External Client App* (nicht eine klassische Connected App).
External Client Apps unterstützen keinen Browser-basierten OAuth-Flow von externen Domains.

**Bereits getestet:**
- `POST /services/oauth2/authorize` (Standard-Endpoint) → `invalid_client_id`
- `POST /services/auth/oauth2/authorize` (External Client App Endpoint) → `Bad_Id`
- Device Flow (`grant_type=device`) → `invalid device code`
- CORS für OAuth-Endpoints aktiviert ✓
- Distribution State auf "Packaged" gesetzt ✓

**Was gebraucht wird:**
Eine klassische Salesforce **Connected App** mit:
- OAuth Flow: Authorization Code + PKCE
- Callback URL: `https://wolfster.github.io/qwello-site-import/callback.html`
- Scopes: `api`, `id`

**Warum nicht selbst erstellt:** Die Salesforce-Permission "Customize Application" fehlt im Account.
Ein externer Salesforce-Entwickler wurde kontaktiert.

**Consumer Key (aktuell, External Client App):**
`3MVG92fMZeJeHJdodQ_Lga8Dk2NnI9BeHVD2dMF7By6.4pSxBcLtlpFtSla3sFs7XS4rAwltO3VZW8tCsGn3`

---

## Nächste Schritte

1. **[BLOCKIERT]** Connected App von Salesforce-Entwickler erstellen lassen
2. Neuen Consumer Key in `index.html` (CONFIG-Block, Zeile ~30) und `callback.html` (Zeile ~39) eintragen
3. End-to-End-Test mit echten Salesforce-Daten
4. Ggf. weitere Länder für UNLOCODE Autocomplete ergänzen

---

## Dateistruktur

```
index.html          — Haupt-App (alles in einer Datei)
callback.html       — OAuth Callback Handler
unlocode-de.js      — UNLOCODE Daten Deutschland (10.009 Einträge)
unlocode-fr.js      — UNLOCODE Daten Frankreich
unlocode-pl.js      — UNLOCODE Daten Polen
unlocode-dk.js      — UNLOCODE Daten Dänemark
unlocode-se.js      — UNLOCODE Daten Schweden
unlocode-es.js      — UNLOCODE Daten Spanien
unlocode-gb.js      — UNLOCODE Daten UK
```
