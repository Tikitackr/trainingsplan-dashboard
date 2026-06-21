# CLAUDE.md — Trainingsplan-Dashboard

Anweisung für Claude in genau diesem Projekt. Ergänzt die globale
`~/.claude/CLAUDE.md`. Wenn Thomas hier eine neue Session öffnet, weißt du
damit sofort, was Sache ist und was bei einem neuen Ausdruck zu tun ist.

## Was das ist
Ein **mobiles Terminplan-Dashboard** (iPhone-15-optimiert) für Thomas'
Reha-Trainingsplan. Eine einzige self-contained `index.html` (CSS + JS inline,
kein Internet/keine Abhängigkeiten) im Liquid-Glass-Stil.

- **Live-URL (GitHub Pages):** https://tikitackr.github.io/trainingsplan-dashboard/
- **Repo:** https://github.com/Tikitackr/trainingsplan-dashboard (öffentlich)
- **Lokal:** `~/Projekte/Trainingsplan/`

## Funktionen
- Tag-Umschalter (Mo/Di/Mi/Do …), aktueller Tag wird automatisch vorgewählt.
- Termine als Karten: Antippen = erledigt (durchgestrichen + gedimmt).
  Häkchen werden pro `Datum|Zeit|Name` in `localStorage` gespeichert → bleiben
  bei einem neuen Ausdruck erhalten, solange Datum/Zeit/Name gleich bleiben.
- Raum-Button pro Termin → öffnet eine **automatisch gezeichnete
  Orientierungskarte** (SVG) als Vollbild-Overlay: der Ziel-Trakt leuchtet.
- Hell/Dunkel-Umschalter, Fortschrittsanzeige pro Tag.

## WICHTIG: Privatsphäre (Repo ist öffentlich!)
Es dürfen **nur** Zeit, Leistung und Raum rein. NIE in die Datei / ins Repo:
Name, Pat.-Nr., Diagnose, Zimmer, Aufnahme-/Entlassungsdatum, Klinikname,
Barcode, Medikamenten-/Gesundheitshinweise (z. B. Marcumar/Peakflow).
Praktische „Mitbringen"-Hinweise (Handtuch, wetterfeste Kleidung, Therapieheft)
sind ok. Im Zweifel weglassen, niemals erfinden.

## So aktualisierst du bei einem NEUEN Ausdruck
Thomas liefert einen Scan/Foto des neuen Therapieplans. Dann:

1. **Lesen:** Termine vom Scan ablesen (Zeit, Leistung, Raum, ggf. Hinweis).
   Privatdaten (siehe oben) ignorieren.
2. **Datenblock anpassen:** In `index.html` NUR den Block `var PLAN = { … }`
   (klar markiert zwischen „DATENBLOCK" und „ENDE DATENBLOCK") bearbeiten.
   CSS/JS nicht anfassen.
   - `meta.woche` auf den neuen Anzeige-Zeitraum setzen.
   - `meta.stand` = heutiges Datum, `meta.quelle` = Ausdruck-Datum.
   - Pro Tag ein Eintrag in `tage[]`: `datum` (ISO "YYYY-MM-DD"), `dow`
     (Mo/Di/…), `termine[]` mit `zeit` ("HH:MM"), `name`, `raum`, optional `note`.
   - **Tab-Leiste ist fest Mo–Sa.** Nur Tage MIT Terminen in `tage[]` eintragen;
     leere Tage (z. B. Fr/Sa) erscheinen automatisch mit „Keine Termine" — nicht
     extra anlegen. Die Datumszahlen der leeren Tage werden aus dem Montag berechnet.
   - **Schlüssel-Stabilität:** Häkchen hängen an `datum|zeit|name`. Ändert sich
     bei einem unveränderten Termin nichts an diesen drei Feldern, bleibt das
     Häkchen erhalten. Geänderte/neue Termine starten leer — so gewollt.
3. **Orientierungskarte:** nichts extra zu tun — sie wird automatisch aus dem
   `raum`-Feld gezeichnet (siehe unten). Wichtig nur: das `raum`-Format
   einhalten, damit Trakt + Ebene erkannt werden.
4. **Lokal prüfen:** `open index.html` und kurz durchklicken.
5. **Committen + pushen** (erst nach Thomas' Freigabe — er ist Dirigent):
   `git add -A && git commit -m "Plan aktualisiert: <Zeitraum>" && git push`
   GitHub Pages baut automatisch neu (~1 Min). Thomas lädt die URL neu.

## Orientierungskarte (automatisch gezeichnet, SVG)
Es gibt **keine Bild-Pläne mehr**. Der Raum-Button öffnet eine schematische
Klinik-Karte im Dashboard-Stil, die per JavaScript gezeichnet wird (Funktion
`mapSVG` in `index.html`). Der **Trakt des Termins leuchtet** in seiner
Bereichsfarbe (Glow + Ziel-Pin), der Rest ist gedimmt; Baustelle und
Haupteingang sind fest eingezeichnet. Darunter Ebenen-Pille + Ziel-Chip + Text.

**Wie die Karte gesteuert wird — nur das `raum`-Feld zählt:**
- Trakt: `Trakt A`…`Trakt F` oder `Kesselhaus` (→ Block „K").
- Ebene: aus `/E0`, `/E1`, `/E-1`; `EG` = Ebene 1; `Trakt A/E · 2/200` = Ebene 2.
- Bereich (Text unter der Karte): alles nach dem Trakt-Code, reine Nummern
  (`5300`, `2/200`) werden rausgefiltert.
- Beispiele: `"Trakt E/E0 · Therapiebereich 3 · Sporthalle"` → E, Ebene 0;
  `"Kesselhaus · EG Vortragssaal"` → K, Ebene 1.

**Karten-Geometrie ändern** (nur wenn ein Trakt umziehen muss): im Array
`var TRAKTE = [ … ]` die Koordinaten anpassen. Anordnung folgt der Vorlage
(Lageplan/Wegweiser, archiviert in `Archiv/`). Spielwiese: `mockup-karte.html`.

## Räume aus dem ersten Ausdruck (Trakt → Ebene → Bereich)
- Trakt E/E0 · Therapiebereich 3 · Sporthalle
- Trakt C/E0 · Therapiebereich 1
- Trakt D/E0 · Therapiebereich 2 · Gymn.halle
- Kesselhaus · EG Vortragssaal
- Trakt F/E1 · 5300 · Schulungsraum 6
- Trakt A/E · 2/200 · Station 2

## Konventionen
- Oberflächensprache Deutsch.
- Keine externen Abhängigkeiten, alles inline (muss offline auf dem iPhone gehen).
- Kritische Aktionen (push) nur mit Thomas' Freigabe.
