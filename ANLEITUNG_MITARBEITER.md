# Anleitung: Qwello Site Import Tool

Mit diesem Tool kannst du mehrere Standorte auf einmal in SiteTracker importieren — ohne Salesforce-Kenntnisse.

---

## Schritt 0: Vorbereitung (einmalig)

Du brauchst:
- ✅ Chrome als Browser
- ✅ Die Chrome Extension **"Salesforce Inspector Reloaded"** installiert  
  → Falls nicht: IT fragen, die können das in 2 Minuten einrichten

---

## Schritt 1: Einloggen

**1.1** Öffne Salesforce in Chrome (wie gewohnt).

**1.2** Klicke auf das **⚡ Salesforce Inspector** Icon oben rechts in Chrome  
*(kleines gelbes Blitz-Symbol in der Symbolleiste)*

**1.3** Im Popup das auf **"Copy"** neben "Session Id" klicken.  
→ Der Token ist jetzt kopiert (du siehst nichts, aber er ist in der Zwischenablage)

**1.4** Tool öffnen: `https://wolfster.github.io/qwello-site-import/`

**1.5** In das Feld "Session Token" klicken → **Strg+V** (Windows) oder **Cmd+V** (Mac) → **Verbinden** klicken

> ⚠ Der Token läuft nach ca. 2 Stunden ab. Wenn das Tool meldet "Token ungültig" → einfach Schritt 1.2–1.5 wiederholen.

---

## Schritt 2: Global Settings ausfüllen

Diese Felder gelten für **alle** Standorte im Import:

| Feld | Was eintragen |
|------|--------------|
| **Batch Name** | Frei wählbar, z.B. "München April 2026" |
| **Country** | Land aus der Liste wählen |
| **Unicode ID** | Stadtcode eingeben oder Stadt tippen und aus Liste wählen |
| **Program** | Programm aus der Liste wählen |
| **Sales Manager** | Person aus der Liste wählen |
| **Project Manager** | Person aus der Liste wählen |

> Programm nicht in der Liste? Wolf bescheid geben — er aktualisiert die Liste.

→ **Continue** klicken

---

## Schritt 3: Daten einfügen

**3.1** Öffne deine Excel-Tabelle mit den Standortdaten.

**3.2** Markiere die Zeilen (inkl. Kopfzeile) → Kopieren (**Strg+C** / **Cmd+C**)

**3.3** In das graue Feld im Tool klicken → Einfügen (**Strg+V** / **Cmd+V**)

**3.4** Anzahl erkannter Zeilen prüfen → **Parse Data** klicken

---

## Schritt 4: Spalten zuordnen

Das Tool erkennt die Spalten meist automatisch.  
Grüne Felder = erkannt ✓ — Rote Felder = muss manuell zugeordnet werden.

→ **Continue** klicken

---

## Schritt 5: Prüfen & Importieren

- Tabelle mit allen Zeilen erscheint
- Rote Zeilen = Fehler (z.B. fehlende Pflichtfelder) — kurz prüfen und korrigieren
- **Import** klicken → Bestätigen

Das Tool zeigt danach für jede Zeile: ✓ Erfolgreich oder ✗ Fehler (mit Grund)

---

## Häufige Probleme

| Problem | Lösung |
|---------|--------|
| "Token ungültig" | Neuen Token aus SF Inspector kopieren (Schritt 1.2–1.4) |
| Programm nicht in der Liste | Wolf fragen — er aktualisiert die Programmliste |
| Spalte wird nicht erkannt | Spaltenname in Excel umbenennen oder manuell zuordnen |
| Import schlägt fehl | Fehlermeldung lesen — meist fehlendes Pflichtfeld |
