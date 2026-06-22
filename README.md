# Trainingsplan-Dashboard

Mobiles Terminplan-Dashboard (iPhone-optimiert) im Liquid-Glass-Stil.
Eine self-contained `index.html` — kein Build, keine Abhängigkeiten.

**Live:** https://tikitackr.github.io/trainingsplan-dashboard/

## Funktionen
- Tag-Umschalter, aktueller Tag automatisch vorgewählt
- Termine abhaken → durchgestrichen, Zustand bleibt (localStorage)
- Heute-Karte mit Live-Uhr, laufendem/nächstem Termin, Essens-Status und Stand-Datum
- Termin-Zustände am Heute-Tag farblich markiert: **jetzt** (grün), **als Nächstes**
  (blau), **überfällig** (bernstein) — in Liste und Zeitband
- Zeitband (Tagesüberblick): voller Punkt = offen/geplant, gedimmter Ring = erledigt,
  bernsteinfarbener Punkt = überfällig
- Raum antippen → automatisch gezeichnete Orientierungskarte (Ziel-Trakt leuchtet)
- Hell/Dunkel-Umschalter, Fortschritt pro Tag
- Belohnungs-Effekte: Konfetti beim Abhaken eines Termins, Feuerwerk + „Tag geschafft"-
  Einblendung bei komplettem Tag (respektiert „Bewegung reduzieren")

## Aktualisieren
Live-Termine stehen in `plan.json` (wird Cache-sicher frisch geladen);
`index.html` hält eine Kopie als Offline-Fallback. Neuen Scan in
`Update-Termine/` ablegen, Ablauf siehe `CLAUDE.md`.
