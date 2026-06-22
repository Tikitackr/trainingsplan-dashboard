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
Thomas legt einen neuen Scan in **`Update-Termine/`** ab (oder reicht ihn direkt).
Dann:

1. **Lesen:** Termine vom Scan in `Update-Termine/` ablesen (Zeit, Leistung,
   Raum, ggf. Hinweis). Privatdaten (siehe oben) ignorieren.
2. **Daten anpassen — `plan.json` ist die Quelle.** Termine in `plan.json`
   eintragen. Dann denselben Stand in den Fallback-Block `var PLAN_DEFAULT = {…}`
   in `index.html` spiegeln (klar markiert „DATENBLOCK"/„ENDE DATENBLOCK").
   **Beide synchron halten.** CSS/JS sonst nicht anfassen.
   - `meta.woche` = neuer Anzeige-Zeitraum; `meta.stand` = heute;
     `meta.quelle` = Ausdruck-Datum.
   - Pro Tag ein Eintrag in `tage[]`: `datum` (ISO "YYYY-MM-DD"), `dow`
     (Mo/Di/…), `termine[]` mit `zeit` ("HH:MM"), `name`, `raum`, optional `note`.
   - **Tab-Leiste ist fest Mo–Sa.** Nur Tage MIT Terminen eintragen; leere Tage
     (z. B. Fr/Sa) erscheinen automatisch mit „Keine Termine". Datumszahlen der
     leeren Tage werden aus dem Montag berechnet.
   - **Schlüssel-Stabilität:** Häkchen hängen an `datum|zeit|name`. Bleiben diese
     drei Felder gleich, bleibt das Häkchen erhalten. Neue/geänderte Termine
     starten leer — so gewollt.
3. **Orientierungskarte:** nichts extra — wird automatisch aus dem `raum`-Feld
   gezeichnet (siehe unten). Nur das `raum`-Format einhalten.
4. **Lokal prüfen:** `open index.html` zeigt den Fallback (file:// lädt kein
   `plan.json`). Für den echten Frisch-Lade-Pfad kurz einen Server starten:
   `python3 -m http.server 8765` → http://localhost:8765/ .
5. **Scan archivieren:** abgearbeiteten Scan von `Update-Termine/` nach
   `Termine/<datum>_Ausdruck/` verschieben.
6. **Committen + pushen** (erst nach Thomas' Freigabe — er ist Dirigent):
   `git add -A && git commit -m "Plan aktualisiert: <Zeitraum>" && git push`
   GitHub Pages baut automatisch neu (~1 Min). Das iPhone zieht `plan.json`
   beim nächsten Öffnen frisch (Cache-Bust), neue Termine erscheinen automatisch.

## Daten-Laden & Cache (warum Updates ohne Cache-Löschen ankommen)
- **`plan.json`** = Live-Termine. Die App lädt sie beim Start frisch mit
  `plan.json?t=<zeit>` und `cache:'no-store'` → umgeht den Browser-Cache gezielt.
- **`PLAN_DEFAULT`** in `index.html` = Offline-/file://-Fallback. Wird vom frisch
  geladenen `plan.json` überschrieben, sobald online.
- **Offline:** der zuletzt geladene Plan liegt im localStorage und wird sofort
  angezeigt; ist kein Netz da, bleibt dieser Stand.
- **Häkchen liegen im localStorage**, getrennt vom Cache. „Cache leeren" löscht
  sie NICHT — nur „Verlauf und Websitedaten löschen" (iOS Safari komplett) würde
  sie löschen. Darum nie zum Aktualisieren die Websitedaten löschen.

## Heute-Karte (Uhr + Zeitband) & Essenszeiten
Unter dem Tageskopf sitzt die Heute-Karte (`renderTodayCard` in `index.html`):
- **Am heutigen Tag** voll: Live-Uhr (Minutentakt via `setInterval`), „jetzt"-
  Termin, Essens-Status („Mittag bis 12:45"), „Als Nächstes … in X h Y", plus
  Zeitband mit Essens-Segmenten, Termin-Punkten (in Trakt-Bereichsfarben) und
  „jetzt"-Linie.
- **An anderen Tagen** nur das Zeitband als „Tagesüberblick" (keine Uhr/jetzt).
- **„als Nächstes dran":** Am heutigen Tag bekommt der nächste *noch offene* Termin
  in der Liste einen Akzent-Rahmen + „als Nächstes"-Pill (`.appt.next`). Auswahl
  via `nextKeyToday()` (erster Termin mit Zeit > jetzt und nicht abgehakt),
  Anwendung über `updateNextHighlight()`; wandert im Minutentakt und beim Abhaken
  weiter (DOM bleibt erhalten, kein Re-Render der Liste).
- **Stand im Kopf:** Die Wochenzeile zeigt zusätzlich „· Stand TT.MM." aus
  `meta.stand` (Footer trägt weiterhin den vollen „Stand … · Ausdruck …"-Text).
- **Erledigte Termine im Zeitband:** ein abgehakter Termin erscheint als
  *gedimmter, hohler Ring* in Trakt-Farbe statt als vollem Punkt (offene Termine
  bleiben voll). Bewusst NICHT ganz ausgeblendet, damit der Tagesüberblick (Form
  des Tages, „jetzt"-Linie im Verhältnis) erhalten bleibt. Steuerung: Done-Check
  via `key()`/`store` in `renderTodayCard`, Optik über CSS `.rdot.done`
  (`opacity` dort anpassen, falls zu blass/kräftig). Abhaken in der Liste ruft
  `renderTodayCard()` → der Ring reagiert sofort.
- **Essenszeiten** stehen in `plan.json` unter `meta.essenszeiten` (`werktag` und
  `wochenende`), gespiegelt im `PLAN_DEFAULT`. Sa/So nutzt den Wochenend-Satz
  (Feiertage werden technisch wie Wochenende behandelt). Ändern sich die
  Essenszeiten, hier anpassen (beide Stellen synchron). Fallback `ESSEN_DEFAULT`
  im Code greift, falls das Feld fehlt.
- Kollision = Termin-Startzeit liegt in einem Essensfenster (Termine haben keine
  Dauer). Das Zeitband zeigt die Überschneidung optisch.

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
(Lageplan/Wegweiser, archiviert in `Archiv/`). Spielwiese: `Archiv/Mockups/mockup-karte.html`.

## Räume aus dem ersten Ausdruck (Trakt → Ebene → Bereich)
- Trakt E/E0 · Therapiebereich 3 · Sporthalle
- Trakt C/E0 · Therapiebereich 1
- Trakt D/E0 · Therapiebereich 2 · Gymn.halle
- Kesselhaus · EG Vortragssaal
- Trakt F/E1 · 5300 · Schulungsraum 6
- Trakt A/E · 2/200 · Station 2

## Konventionen
- Oberflächensprache Deutsch.
- Keine externen Abhängigkeiten/CDNs. CSS+JS inline in `index.html`; einzige
  lokale Datei daneben ist `plan.json` (eigene Domain, Offline-Fallback eingebaut).
- Kritische Aktionen (push) nur mit Thomas' Freigabe.
