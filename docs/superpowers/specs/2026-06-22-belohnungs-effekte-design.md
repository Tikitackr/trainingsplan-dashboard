# Belohnungs-Effekte (Konfetti & Feuerwerk) — Design

**Datum:** 2026-06-22
**Projekt:** Trainingsplan-Dashboard
**Status:** freigegeben, bereit für Umsetzungsplan

## Ziel
Spielerische Belohnungsmomente fürs Reha-Dashboard:
- **Konfetti** beim Abhaken eines einzelnen Termins.
- **Feuerwerk** + Glückwunsch-Einblendung, wenn ein Tag komplett erledigt ist.

Gewählte Intensität: **„Voll feiern"** (Variante B aus dem Mockup) — Vollbild-Konfettiregen,
Feuerwerk mit mehreren Raketen quer über die Seite.

## Rahmenbedingungen (Projekt-Invarianten)
- **Self-contained:** alles inline in `index.html` (CSS + JS). Keine Library, kein CDN,
  kein Build. `plan.json` bleibt unberührt.
- **Öffentliches Repo:** keine Privatdaten betroffen — rein visuell.
- **iPhone-Ziel:** Effekte müssen flüssig laufen und den Akku schonen.
- **Bestehende Konvention:** `prefers-reduced-motion` wird respektiert (Projekt nutzt das
  bereits, z. B. bei der Karten-Pulse-Animation).

## Architektur

### 1. Effekt-Engine (neu, inline in `index.html`)
- Ein **Canvas-Overlay** `<canvas id="fx">`: `position:fixed; inset:0; pointer-events:none`,
  hoher z-index über dem Dashboard. Größe an `devicePixelRatio` (Deckel 2) angepasst,
  `resize`-Handler.
- Ein **Partikel-Array** und **ein** `requestAnimationFrame`-Loop. Der Loop läuft nur,
  solange Partikel existieren, und **beendet sich selbst**, sobald das Array leer ist
  (kein Dauer-rAF → kein Akku-Dauerverbrauch).
- Zwei Partikel-Erzeuger:
  - `confettiRain()` — ~140 rechteckige Konfetti-Partikel, fallen von oben über den Schirm,
    leichte Drift + Rotation, Lebensdauer ~1,5 s.
  - `firework(x, y)` — radiale Funken-Explosion (~70 Funken) in einer Farbe; wird für den
    Tagesabschluss mehrfach an verschiedenen Positionen leicht zeitversetzt gezündet.
- **Partikel-Obergrenze** (~400 aktiv): neue Erzeuger fügen nichts hinzu, was den Deckel
  überschreitet, damit schnelles Mehrfach-Abhaken nicht ruckelt.

### 2. Glückwunsch-Einblendung (Toast)
- Festes `.toast`-Element „🎉 Tag geschafft!", kurz eingeblendet (~2,2 s) beim Tagesabschluss.

### 3. Auslöser (im vorhandenen Karten-Handler `index.html:558`)
Der Handler kennt bereits `now` (= wird gerade abgehakt) und ruft `store.set`, `updateProgress`,
`renderTodayCard`, `updateTodayStates`. Erweiterung:
- **Vor** `store.set`: `dayDone(tag)` als `wasDone` merken.
- `store.set` wie bisher.
- Wenn `now === true` (abgehakt, nicht aufgehoben): **Konfetti** zünden.
- **Nach** dem Setzen: `dayDone(tag)` erneut prüfen → `isDone`.
  Wenn `!wasDone && isDone`: **Feuerwerk + Toast** (genau am Übergang offen→komplett).
- Beim Aufheben (`now === false`): kein Effekt.

### 4. Reduced-Motion-Pfad
- Wird `prefers-reduced-motion: reduce` erkannt, ersetzt ein dezenter Statt-Moment die
  Partikel: kurzes Aufleuchten der Karte beim Abhaken; beim Tagesabschluss nur der Toast,
  kein Feuerwerk. Geprüft über `window.matchMedia('(prefers-reduced-motion: reduce)')`.

## Datenfluss
Tap auf Karte → Handler: `wasDone` merken → `store.set` → (abgehakt?) Konfetti →
bestehende Re-Render-Aufrufe → (Übergang auf komplett?) Feuerwerk + Toast.
Kein neuer Zustand persistiert; Effekte sind rein flüchtig. Häkchen-Logik (localStorage)
unverändert.

## Verhalten / Edge Cases
- Effekte greifen an **jedem** Tag, nicht nur am heutigen — Abhaken bleibt überall konsistent.
- Tag bereits komplett → einen Termin aufheben → wieder abhaken: Feuerwerk feuert erneut
  (erneuter Übergang offen→komplett, gewollt).
- Schnelles Mehrfach-Abhaken: mehrere Konfetti-Wellen, durch Partikel-Deckel begrenzt.
- Mehrfach-Tap aufs selbe (toggle on/off): nur die „on"-Übergänge feuern.

## Bewusst NICHT dabei
- Keine Vibration/Haptik (iOS Safari unterstützt die Vibration-API nicht — nicht erfinden).
- Kein Sound.
- Keine Einstellungsoption zum Abschalten (über `prefers-reduced-motion` abgedeckt).

## Betroffene Dateien
- `index.html` — neues `<canvas>` + `.toast`-Markup, CSS für beide, Effekt-Engine-JS,
  Erweiterung des Karten-Handlers. Nur additive Änderungen; bestehende Logik unberührt.
- `CLAUDE.md` / `README.md` — kurzer Hinweis auf die Effekte (Doku-Konvention des Projekts).

## Verifikation
- Lokal via `python3 -m http.server` öffnen (echter Lade-Pfad mit `plan.json`).
- Einzeltermin abhaken → Konfettiregen, beim Aufheben nichts.
- Alle Termine eines Tages abhaken → genau beim letzten das Feuerwerk + Toast.
- Bei aktiviertem „Bewegung reduzieren" (macOS/iOS) → kein Geböller, nur Statt-Moment.
- Flüssig auf dem iPhone (kein Ruckeln bei schnellem Abhaken).
