# Qwello Site Import Tool — Developer Prompt

## Projektziel

Internes Web-Tool für das Qwello-Team, mit dem Mitarbeiter ohne Salesforce-Kenntnisse neue Standorte (Sites) direkt in SiteTracker (Salesforce-basiert) anlegen können — über ein geführtes Formular mit validiertem CSV-Export für den Import via **Salesforce Inspector Reloaded**.

---

## Stack & Hosting

- **Reines HTML/CSS/JS** — kein Framework, kein Build-Step, keine Dependencies
- **Hosting:** GitHub Pages unter `https://wolfster.github.io/qwello-site-import`
- **Zwei Dateien:**
  - `index.html` — Haupt-App mit Login, Formular, CSV-Export
  - `callback.html` — OAuth Callback-Handler

---

## Authentifizierung

**Salesforce OAuth 2.0 mit PKCE** (kein Client Secret im Frontend)

| Parameter | Wert |
|---|---|
| Salesforce Domain | `https://sitetracker-qwello.my.salesforce.com` |
| Connected App Name | `Qwello Site Import` (External Client App) |
| Client ID (Consumer Key) | `3MVG92fFMZeJeHJdodQ_Lga8Dk2Nnl9BeHVD2dMF7By6.4pSxBcLtlpFtSla3sFs7XS4rAwltO3VZW8tCsGn3` |
| Redirect URI | `https://wolfster.github.io/qwello-site-import/callback.html` |
| OAuth Scopes | `api`, `id` |
| Flow | Authorization Code + PKCE (`S256`) |
| CORS | `https://wolfster.github.io` muss in Salesforce Setup → CORS freigegeben sein |

**Flow:**
1. `index.html` generiert PKCE Code Verifier + Challenge, speichert Verifier in `sessionStorage`
2. Redirect zu Salesforce Login
3. Salesforce redirectet zu `callback.html?code=...`
4. `callback.html` tauscht Code gegen Access Token (POST zu `/services/oauth2/token`)
5. Token + Instance URL werden in `sessionStorage` gespeichert
6. Redirect zurück zu `index.html`

Token liegt nur im `sessionStorage` (wird bei Tab-Schließen gelöscht, nie in `localStorage`).

---

## Salesforce Objekt

**Object API Name:** `sitetracker__Site__c`

---

## Felder & Validierung

### Pflichtfelder (müssen vor CSV-Export ausgefüllt sein)

| Label | API-Name | Typ | Besonderheit |
|---|---|---|---|
| Site Name | `Name` | Text | Frei |
| Program | `Program__c` | Lookup → `sitetracker__Program__c` | Live-Suche, ID wird eingesetzt |
| Site Status (SiteTracker) | `sitetracker__Site_Status__c` | Picklist | Siehe Werte unten |
| Site Status 2026 | `Site_Status_2026__c` | Picklist | Steuert Rejection-Sektion |
| Country | `Country__c` | Picklist | Steuert UNECE-Link |
| City | `City__c` | Text | Frei |
| Unicode ID | `Unicode_ID__c` | Text | Aus UNECE-Liste, Link erscheint nach Länderauswahl |
| Lat | `sitetracker__Lat__c` | Zahl (Decimal) | Breitengrad |
| Long | `sitetracker__Long__c` | Zahl (Decimal) | Längengrad |
| Sales Manager | `Sales_Manager__c` | Lookup → `User` | Live-Suche, ID wird eingesetzt |
| Project Manager | `Project_Manager__c` | Lookup → `User` | Live-Suche, ID wird eingesetzt |

### Optionale Felder

| Label | API-Name | Typ |
|---|---|---|
| Street Address | `sitetracker__Street_Address__c` | Text |
| Zip Code | `sitetracker__Zip_Code__c` | Text |
| State | `State__c` | Text |
| Administrative District | `Administrative_District__c` | Text |
| Region | `Region__c` | Text |
| Location Concept Score | `Location_Concept_Score__c` | Zahl |
| Pole Model | `Pole_Model__c` | Picklist |
| Number of Poles | `Number_of_poles__c` | Zahl |
| Location Design | `Location_Design__c` | Picklist |
| Location Setup | `Location_Setup__c` | Picklist |
| Sidewalk Width | `Sidewalk_width__c` | Zahl (Meter) |
| Parking Space Length | `Parking_space_length__c` | Zahl (Meter) |
| Parking Space Width | `Parking_space_width__c` | Zahl (Meter) |
| Allowed Parking Time | `Allowed_Parking_Time__c` | Text |
| Contract End Date | `Contract_End_Date__c` | Date (YYYY-MM-DD) |

### Rejection-Felder (nur wenn `Site_Status_2026__c` = `Rejected`)

| Label | API-Name | Typ |
|---|---|---|
| Rejection Actor | `Rejection_Actor__c` | Picklist |
| Rejection Reason | `Rejection_Reason__c` | Picklist |
| Rejection Date | `Rejection_Date__c` | Date |
| Rejection Reason Description | `Rejection_Reason_Description__c` | Text (Textarea) |

---

## Picklist-Werte (Stand April 2025, aus Salesforce gezogen)

### `sitetracker__Site_Status__c`
`Imported and Reserve`, `Live`, `In Contract Negotiation`, `Built Up - Not Live`, `Under Contract`, `Rejected`, `Location Concept`, `All Permits on Hand`, `Backlog`, `Decommissioned`, `PowerRacing`, `Submitted`, `Open`

### `Site_Status_2026__c`
`Live`, `Rejected`, `Development`, `Live & Under Development`, `Evaluation`, `Decommissioned`

### `Country__c`
`Denmark`, `Spain`, `Sweden`, `UK`, `France`, `Germany`, `Poland`

### `Pole_Model__c`
`CP05`, `CP07`, `CP21`, `CP06`, `Ecotap Duo Wide`, `Autel DC50`, `Autel DH240`, `tba`, `CP22`

### `Location_Design__c`
`Perpendicular`, `Longitudinal`, `Diagonal`

### `Location_Setup__c`
`Covered Car Park`, `Other`, `Island`, `Pavement`, `Car Park`

### `Rejection_Actor__c`
`City`, `Qwello`, `Grid Provider`, `City District`

### `Rejection_Reason__c`
`Parking restrictions`, `Bicycle Road`, `Tree Protection`, `Narrow Parkingspace`, `Monument Protection`, `Narrow Pavement`, `Grid Capacity`, `Other`, `Other Operator`

---

## Live-Lookup-Felder (Salesforce REST API)

Für `Program__c`, `Sales_Manager__c`, `Project_Manager__c` wird die Salesforce REST API abgefragt. Der Nutzer tippt einen Namen, Ergebnisse erscheinen als Dropdown. Nach Auswahl wird der **lesbare Name** im Input angezeigt, die **Salesforce ID** in einem Hidden Field gespeichert — nur die ID geht in die CSV.

**SOQL für Program:**
```
SELECT Id, Name FROM sitetracker__Program__c WHERE Name LIKE '%{query}%' ORDER BY Name LIMIT 10
```

**SOQL für User (Sales/PM):**
```
SELECT Id, Name, Email FROM User WHERE (Name LIKE '%{query}%' OR Email LIKE '%{query}%') AND IsActive = true ORDER BY Name LIMIT 10
```

**API-Endpoint:**
```
GET {instanceUrl}/services/data/v59.0/query?q={encodedSOQL}
Authorization: Bearer {accessToken}
```

Debounce: 350ms. Mindestlänge: 2 Zeichen.

---

## Unicode ID — UNECE-Logik

Nach Länderauswahl erscheint ein klickbarer Link zur offiziellen UNECE-Locode-Liste des Landes. Der Nutzer schlägt den Code nach und trägt ihn manuell ein.

| Land | UNECE URL |
|---|---|
| Germany | `https://service.unece.org/trade/locode/de.htm` |
| Poland | `https://service.unece.org/trade/locode/pl.htm` |
| Denmark | `https://service.unece.org/trade/locode/dk.htm` |
| Sweden | `https://service.unece.org/trade/locode/se.htm` |
| Spain | `https://service.unece.org/trade/locode/es.htm` |
| France | `https://service.unece.org/trade/locode/fr.htm` |
| UK | `https://service.unece.org/trade/locode/gb.htm` |

---

## CSV-Export

- **Format:** Komma-separiert, UTF-8, erste Zeile = API-Feldnamen
- **Inhalt:** Nur befüllte Felder — leere optionale Felder werden weggelassen
- **Lookup-Felder:** Nur die Salesforce-ID (z.B. `001...`), nicht der Name
- **Datums-Felder:** Format `YYYY-MM-DD`
- **Dateiname:** `site-import_{SiteName}_{YYYY-MM-DD}.csv`

Beispiel-Output:
```csv
Name,Program__c,sitetracker__Site_Status__c,Site_Status_2026__c,Country__c,City__c,Unicode_ID__c,sitetracker__Lat__c,sitetracker__Long__c,Sales_Manager__c,Project_Manager__c
DE-MUC-001,a0X5g000001abcDEF,Location Concept,Development,Germany,München,DEMUC,48.137154,11.576124,0055g000003xyzABC,0055g000003xyzDEF
```

---

## Formular-Logik

- **Rejection-Sektion** ist standardmäßig ausgeblendet — erscheint nur wenn `Site_Status_2026__c = Rejected`
- **Validierung** läuft vor dem CSV-Export: alle Pflichtfelder müssen ausgefüllt sein, Lookup-Felder müssen eine ID haben (nicht nur Text)
- **Fehler** werden inline pro Feld angezeigt, erstes Fehlerfeld wird in den Viewport gescrollt
- **Reset-Button** setzt alle Felder zurück inkl. Hidden Fields und blendet Rejection-Sektion aus
- **Vorschau-Button** zeigt CSV-Inhalt im Guide-Panel an ohne Download

---

## Anleitung für Endnutzer (im Tool eingebaut)

Das Tool enthält eine ein-/ausklappbare Anleitung mit diesen Schritten:
1. Formular ausfüllen → CSV exportieren
2. Salesforce Inspector Reloaded öffnen (Chrome Extension)
3. Object: `sitetracker__Site__c`, Action: `Insert`
4. CSV hochladen
5. Import ausführen
6. Bei Fehler: Screenshot an `wgu@qwello.de`

---

## Sicherheit & Zugang

- Nur Nutzer mit aktivem Qwello Salesforce Account können sich einloggen (OAuth)
- Kein Client Secret im Frontend-Code (PKCE-Flow)
- Token nur in `sessionStorage`, nie in `localStorage`
- GitHub Repo ist public (Voraussetzung für GitHub Pages), aber der Code enthält keine sensiblen Daten außer der Client ID (die ohne Secret wertlos ist)
- CORS muss in Salesforce Setup für `https://wolfster.github.io` freigeschaltet sein

---

## Bekannte offene Punkte / mögliche Erweiterungen

- [ ] CORS-Fehler testen sobald deployed (Salesforce CORS-Eintrag nötig)
- [ ] Callback URL in Connected App auf exakte URL prüfen: `https://wolfster.github.io/qwello-site-import/callback.html`
- [ ] Später: Repo zu Qwello Company GitHub Account umziehen (dann URL anpassen in `index.html`, `callback.html` und Connected App)
- [ ] Später: Picklist-Werte live aus Salesforce laden statt hardcoded (über `PicklistValueInfo` SOQL)
- [ ] Später: Multi-Row-Export (mehrere Sites in einer CSV)
