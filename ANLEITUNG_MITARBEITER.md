# Anleitung: Qwello Site Import Tool

Mit diesem Tool kannst du mehrere Standorte auf einmal in SiteTracker importieren — ohne Salesforce-Kenntnisse.

---

## Schritt 0: Vorbereitung (einmalig)

Du brauchst:
- ✅ Chrome als Browser (kein Firefox, kein Edge)
- ✅ In Salesforce eingeloggt sein

---

## Schritt 1: Einloggen

**1.1** Öffne Salesforce in Chrome und stelle sicher, dass du eingeloggt bist.

**1.2** Drücke die Taste **F12** auf der Tastatur.  
→ Am rechten Rand öffnet sich ein Entwicklerfenster (sieht kompliziert aus, aber du brauchst nur einen einzigen Klick darin).

**1.3** Klicke oben in diesem Fenster auf den Tab **"Application"**.

**1.4** Im linken Bereich: auf **"Cookies"** klicken → darunter auf **"sitetracker-qwello.my.salesforce.com"** klicken.

**1.5** In der Tabelle rechts die Zeile **`sid`** suchen → auf den langen Wert in der Spalte "Value" klicken → **Strg+A** → **Strg+C**.

**1.6** Drücke nochmal **F12** um das Fenster zu schließen.

**1.7** Tool öffnen: `https://wolfster.github.io/qwello-site-import/`  
→ In das Feld "Session Token" klicken → **Strg+V** → **Verbinden** klicken.

> ⚠ Wenn das Tool "Token ungültig" meldet: Schritt 1.2–1.7 wiederholen (neuen Token kopieren).

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
