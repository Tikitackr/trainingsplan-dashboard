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
- Raum-Button pro Termin → öffnet den zugehörigen Grundriss als Vollbild-Overlay.
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
   - **Schlüssel-Stabilität:** Häkchen hängen an `datum|zeit|name`. Ändert sich
     bei einem unveränderten Termin nichts an diesen drei Feldern, bleibt das
     Häkchen erhalten. Geänderte/neue Termine starten leer — so gewollt.
3. **Grundriss-Zuordnung** (falls relevant, siehe unten): bei jedem Termin
   optional `grundriss: "<schlüssel>"` setzen.
4. **Lokal prüfen:** `open index.html` und kurz durchklicken.
5. **Committen + pushen** (erst nach Thomas' Freigabe — er ist Dirigent):
   `git add -A && git commit -m "Plan aktualisiert: <Zeitraum>" && git push`
   GitHub Pages baut automatisch neu (~1 Min). Thomas lädt die URL neu.

## Grundrisse ergänzen (kommen als 2 Scans nach)
1. Scans als Bilder im Projektordner ablegen, z. B. `grundriss-1.png`,
   `grundriss-2.png` (komprimiert, < ~1 MB pro Bild).
2. In `index.html` im Objekt `var GRUNDRISSE = { … }` Einträge anlegen:
   `"kesselhaus": { titel: "Kesselhaus / EG", bild: "grundriss-1.png" }`.
3. Bei den Terminen den passenden Schlüssel als `grundriss: "kesselhaus"`
   eintragen. Räume ohne Zuordnung zeigen „Grundriss folgt".
4. Committen + pushen.

## Räume aus dem ersten Ausdruck (für die Grundriss-Zuordnung)
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
