# Trainingsplan-Dashboard

Mobiles Terminplan-Dashboard (iPhone-optimiert) im Liquid-Glass-Stil.
Eine self-contained `index.html` — kein Build, keine Abhängigkeiten.

**Live:** https://tikitackr.github.io/trainingsplan-dashboard/

## Funktionen
- Tag-Umschalter, aktueller Tag automatisch vorgewählt
- Termine abhaken → durchgestrichen, Zustand bleibt (localStorage)
- Heute-Karte mit Live-Uhr, laufendem/nächstem Termin, Essens-Status und
  Zeitband (Tagesüberblick); erledigte Termine erscheinen im Zeitband als
  gedimmter Ring, offene als voller Punkt
- Raum antippen → automatisch gezeichnete Orientierungskarte (Ziel-Trakt leuchtet)
- Hell/Dunkel-Umschalter, Fortschritt pro Tag

## Aktualisieren
Live-Termine stehen in `plan.json` (wird Cache-sicher frisch geladen);
`index.html` hält eine Kopie als Offline-Fallback. Neuen Scan in
`Update-Termine/` ablegen, Ablauf siehe `CLAUDE.md`.
