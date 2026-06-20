# Trainingsplan-Dashboard

Mobiles Terminplan-Dashboard (iPhone-optimiert) im Liquid-Glass-Stil.
Eine self-contained `index.html` — kein Build, keine Abhängigkeiten.

**Live:** https://tikitackr.github.io/trainingsplan-dashboard/

## Funktionen
- Tag-Umschalter, aktueller Tag automatisch vorgewählt
- Termine abhaken → durchgestrichen, Zustand bleibt (localStorage)
- Raum antippen → Grundriss als Vollbild-Overlay
- Hell/Dunkel-Umschalter, Fortschritt pro Tag

## Aktualisieren
Plan-Daten stehen im Block `var PLAN = { … }` in `index.html`.
Ablauf siehe `CLAUDE.md`.
