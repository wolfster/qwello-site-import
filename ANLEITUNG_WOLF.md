# Tool-Anleitung — Wolf

## Token holen (jedes Mal beim Einloggen)

1. Salesforce in Chrome öffnen
2. **⚡ Salesforce Inspector** Icon in der Chrome-Leiste klicken
3. Im Popup auf **Copy** neben "Session Id" klicken
4. Im Tool einfügen → **Verbinden**

> Token läuft nach ca. 2h ab (Session-abhängig). Einfach neu kopieren.

---

## Cache aktualisieren (monatlich oder vor neuem Programm)

Mit deinem Token eingeloggt: oben rechts **↻ Cache aktualisieren** klicken.  
→ `cache.json` wird heruntergeladen (enthält alle Programme + aktive User).

Dann ins Repo hochladen:

```bash
# cache.json ins Projektverzeichnis schieben, dann:
git add cache.json
git commit -m "Cache update $(date +%Y-%m-%d)"
git push
```

Alle Kollegen sehen die neuen Daten beim nächsten Seitenaufruf.

---

## Neues Programm kurz vor Import

**Option A (empfohlen):** Cache aktualisieren wie oben — sobald das Programm in SF existiert.

**Option B:** Kollege wählt im Program-Dropdown die letzte Option  
→ **"✏ ID manuell eingeben…"** → SF-ID direkt eingeben.  
Die ID steht in der URL wenn man das Programm in Salesforce öffnet:  
`…/sitetracker__Program__c/`**`a0X5g000001XXXXX`**`/view`

---

## Notfall: OAuth-Stand wiederherstellen

```bash
git checkout oauth-backup
```

Branch `oauth-backup` enthält den vollständigen PKCE/Device-Flow Stand (Stand vor Token-Paste Umbau).

---

## Tool-URL

`https://wolfster.github.io/qwello-site-import/`
