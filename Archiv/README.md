# Archiv

Hier liegen ausgemusterte Dateien, die das Dashboard nicht mehr aktiv nutzt,
die aber als Referenz/Vorlage aufbewahrt werden.

## Inhalt

- `grundriss-uebersicht.jpg` — abfotografierter **Lageplan** der Klinik
  (Übersicht aller Bereiche A–K). War bis 2026-06-20 als Overlay im Dashboard
  hinterlegt.
- `grundriss-ebenen.jpg` — abfotografierter **Wegweiser** (die drei Ebenen
  E1 · E0 · E−1). Ebenfalls bis 2026-06-20 als Overlay hinterlegt.
- `Mockups/` — Design-Spielwiesen, mit denen Layout-Varianten vor dem Einbau
  ins Dashboard durchprobiert wurden (am 2026-06-22 hierher verschoben, nachdem
  die finalen Varianten in `index.html` live und mobil getestet waren):
  - `mockup-karte.html` — Orientierungskarte (Vorlage für das Inline-SVG `mapSVG`).
  - `mockup-essenszeiten.html` / `mockup-heute.html` / `mockup-heute-vergleich.html`
    — Konzepte für Heute-Karte + Essenszeiten (gewählt wurde B+E).

## Warum archiviert?

Am 2026-06-21 wurden die abfotografierten PDF-Pläne durch **selbstgezeichnete,
schematische SVG-Karten** im Dashboard-Stil ersetzt (siehe `index.html`,
Funktion `mapSVG`). Diese beiden JPGs dienten als **Vorlage** für die Anordnung
der Trakte und die Bereichsfarben — deshalb hier aufbewahrt, falls die Karte
mal nachgezogen oder erweitert werden muss.

Die Original-Scans (mit Privatdaten) liegen separat in `Eingang/` und sind per
`.gitignore` vom öffentlichen Repo ausgeschlossen. Diese beiden JPGs hier sind
beschnittene, allgemeine Orientierungskarten **ohne** Patientendaten.
