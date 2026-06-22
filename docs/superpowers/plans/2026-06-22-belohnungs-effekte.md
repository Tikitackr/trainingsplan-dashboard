# Belohnungs-Effekte (Konfetti + Feuerwerk) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Beim Abhaken eines Termins regnet Vollbild-Konfetti; ist ein Tag komplett erledigt, zündet ein Feuerwerk mit Glückwunsch-Einblendung.

**Architecture:** Ein fixes Canvas-Overlay über dem Dashboard, eine kleine Inline-Partikel-Engine mit einem selbst-beendenden `requestAnimationFrame`-Loop. Die Effekte werden im bestehenden Karten-Klick-Handler ausgelöst; der Tagesabschluss wird über einen Vorher/Nachher-Vergleich von `dayDone()` erkannt.

**Tech Stack:** Reines HTML5 Canvas + Vanilla-JS, alles inline in `index.html`. Keine Library, kein CDN, kein Build.

## Global Constraints

- **Self-contained:** alle Änderungen inline in `index.html` (CSS + JS). Keine externen Abhängigkeiten, kein CDN, kein Build.
- **`plan.json` bleibt unberührt.** Datenformat und Häkchen-Logik (localStorage) nicht anfassen.
- **Öffentliches Repo:** keine Privatdaten in Code/Doku — Effekte sind rein visuell.
- **`prefers-reduced-motion: reduce` respektieren** (Projekt-Konvention, siehe vorhandene `@media`-Regel ~`index.html:61`).
- **Oberflächensprache Deutsch.**
- **iPhone-Ziel:** flüssig, akkuschonend — rAF-Loop läuft nur bei aktiven Partikeln.
- **Kein Push ohne Thomas' Freigabe.** Commits lokal sind ok.
- **Verifikation = manuelle Browser-Prüfung** (Projekt hat kein Test-Framework; keines einführen). Lokaler Server: `python3 -m http.server 8765` → http://localhost:8765/ .

---

### Task 1: Canvas-Overlay + Partikel-Engine

**Files:**
- Modify: `index.html` — neues `<canvas id="fx">` + `.toast`-Markup im `<body>`; CSS dafür; Engine-JS im bestehenden `<script>`.

**Interfaces:**
- Consumes: nichts.
- Produces: globale Funktionen `confettiRain()`, `firework(x, y)`, `celebrateToast()`, sowie internes `parts`-Array, `run()` (startet Loop) und Konstante `MAX_PARTS = 400`. Reduced-Motion-Flag `REDUCE` (boolean).

- [ ] **Step 1: Canvas- und Toast-Markup einfügen**

Direkt nach dem öffnenden `<body>`-Tag in `index.html` einfügen:

```html
<canvas id="fx"></canvas>
<div class="toast" id="celebrate">🎉 Tag geschafft!</div>
```

- [ ] **Step 2: CSS ergänzen**

Im `<style>`-Block (z. B. ans Ende, vor `</style>`) einfügen:

```css
  #fx { position: fixed; inset: 0; width: 100vw; height: 100vh; pointer-events: none; z-index: 90; }
  .toast { position: fixed; left: 50%; top: 16%; transform: translateX(-50%) translateY(-12px);
    background: rgba(20,28,52,.92); border: 1px solid var(--stroke, rgba(255,255,255,.12));
    color: #fff; padding: 11px 18px; border-radius: 999px; font-weight: 600; font-size: 15px;
    z-index: 95; opacity: 0; transition: .3s; backdrop-filter: blur(10px);
    box-shadow: 0 8px 30px rgba(0,0,0,.4); pointer-events: none; }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }
```

- [ ] **Step 3: Engine-JS einfügen**

Im bestehenden `<script>` (vor dem schließenden `})();` / am Ende der IIFE, jedenfalls im selben Scope wie die übrigen Funktionen) einfügen:

```javascript
  // ---- Belohnungs-Effekte: Canvas-Partikel-Engine ----
  var REDUCE = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  var fxCv = document.getElementById('fx'), fxCtx = fxCv.getContext('2d');
  var FXDPR = Math.min(2, window.devicePixelRatio || 1);
  function fxSize(){ fxCv.width = innerWidth * FXDPR; fxCv.height = innerHeight * FXDPR; fxCtx.setTransform(FXDPR,0,0,FXDPR,0,0); }
  fxSize(); window.addEventListener('resize', fxSize);
  var parts = [], fxOn = false, MAX_PARTS = 400;
  var FXCOL = ['#6ea8fe','#a78bfa','#34d399','#fbbf24','#f472b6','#22d3ee'];
  function fxColor(){ return FXCOL[Math.floor(Math.random() * FXCOL.length)]; }
  function fxTick(){
    fxCtx.clearRect(0,0,innerWidth,innerHeight);
    for (var i = parts.length - 1; i >= 0; i--){
      var p = parts[i];
      p.vy += p.g; p.x += p.vx; p.y += p.vy; p.vx *= 0.99; p.life--; p.rot += p.vr;
      fxCtx.globalAlpha = Math.max(0, Math.min(1, p.life / p.fade));
      if (p.kind === 'spark'){ fxCtx.fillStyle = p.c; fxCtx.beginPath(); fxCtx.arc(p.x, p.y, p.r, 0, 7); fxCtx.fill(); }
      else { fxCtx.save(); fxCtx.translate(p.x, p.y); fxCtx.rotate(p.rot); fxCtx.fillStyle = p.c; fxCtx.fillRect(-p.r, -p.r*0.6, p.r*2, p.r*1.2); fxCtx.restore(); }
      if (p.life <= 0 || p.y > innerHeight + 40) parts.splice(i, 1);
    }
    fxCtx.globalAlpha = 1;
    if (parts.length){ requestAnimationFrame(fxTick); } else { fxOn = false; fxCtx.clearRect(0,0,innerWidth,innerHeight); }
  }
  function run(){ if (!fxOn){ fxOn = true; requestAnimationFrame(fxTick); } }
  function confettiRain(){
    if (REDUCE) return;
    var n = Math.min(140, MAX_PARTS - parts.length);
    for (var i = 0; i < n; i++){
      parts.push({ x: Math.random()*innerWidth, y: -20 - Math.random()*innerHeight*0.5,
        vx: (Math.random()-0.5)*1.5, vy: 2 + Math.random()*3, g: 0.05,
        r: 3 + Math.random()*4, c: fxColor(), life: 120, fade: 90,
        rot: Math.random()*7, vr: (Math.random()-0.5)*0.5, kind: 'conf' });
    }
    run();
  }
  function firework(x, y){
    if (REDUCE) return;
    var n = Math.min(70, MAX_PARTS - parts.length), base = fxColor();
    for (var i = 0; i < n; i++){
      var ang = (i / n) * Math.PI * 2, sp = 3.5 + Math.random()*2;
      parts.push({ x: x, y: y, vx: Math.cos(ang)*sp, vy: Math.sin(ang)*sp, g: 0.045,
        r: 2 + Math.random()*2, c: Math.random() < 0.7 ? base : fxColor(), life: 70, fade: 70,
        rot: 0, vr: 0, kind: 'spark' });
    }
    run();
  }
  function celebrateToast(){
    var t = document.getElementById('celebrate'); if (!t) return;
    t.classList.add('show'); setTimeout(function(){ t.classList.remove('show'); }, 2200);
  }
```

- [ ] **Step 4: Engine im Browser verifizieren**

Server starten: `python3 -m http.server 8765` (im Projektordner). http://localhost:8765/ öffnen, DevTools-Konsole öffnen.
In der Konsole eingeben: `confettiRain()` → Konfetti fällt; danach `firework(innerWidth/2, innerHeight*0.3)` → Funken-Explosion; `celebrateToast()` → Toast erscheint und verschwindet nach ~2 s.
Erwartet: keine Konsolenfehler; nach Ende der Animation ist `fxOn === false` (in Konsole prüfbar) → Loop hat sich selbst beendet.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Belohnungs-Effekte: Canvas-Overlay + Partikel-Engine (Konfetti/Feuerwerk/Toast)"
```

---

### Task 2: Konfetti beim Abhaken auslösen

**Files:**
- Modify: `index.html` — Karten-Klick-Handler (`index.html:558`, der `card.addEventListener('click', …)`).

**Interfaces:**
- Consumes: `confettiRain()` aus Task 1; vorhandene `now`-Variable im Handler (true = wird gerade abgehakt).
- Produces: nichts Neues.

- [ ] **Step 1: Handler erweitern**

Im Klick-Handler, direkt **nach** `card.classList.toggle('done', now);` und **vor** `updateProgress();`, einfügen:

```javascript
        if (now) confettiRain();
```

Der Handler sieht danach so aus:

```javascript
      card.addEventListener('click', function(){
        var now = store.get(k) !== '1';
        store.set(k, now ? '1' : '0');
        card.classList.toggle('done', now);
        if (now) confettiRain();
        updateProgress();
        renderTodayCard();
        updateTodayStates();
      });
```

- [ ] **Step 2: Im Browser verifizieren**

http://localhost:8765/ neu laden. Einen Termin antippen → Konfetti regnet, Karte wird durchgestrichen. Denselben Termin erneut antippen (Häkchen aufheben) → **kein** Konfetti. Mehrere Termine schnell hintereinander abhaken → mehrere Konfetti-Wellen, kein Ruckeln.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Belohnungs-Effekte: Konfetti beim Abhaken eines Termins"
```

---

### Task 3: Feuerwerk + Toast beim Tagesabschluss

**Files:**
- Modify: `index.html` — derselbe Karten-Klick-Handler.

**Interfaces:**
- Consumes: `firework(x, y)`, `celebrateToast()` aus Task 1; vorhandene Funktion `dayDone(tag)` (`index.html:407`); `tag` ist im Render-Scope des Handlers verfügbar (die Karte wird je `tag` gerendert).
- Produces: lokale Hilfsfunktion `dayFireworks()` im Handler-Scope (mehrere zeitversetzte Raketen).

- [ ] **Step 1: Übergangs-Erkennung in den Handler einbauen**

Den Handler so erweitern: `wasDone` **vor** `store.set` erfassen, `isDone` **nach** den Re-Render-Aufrufen prüfen, und beim Übergang das Feuerwerk zünden. Vollständiger Handler:

```javascript
      card.addEventListener('click', function(){
        var wasDone = dayDone(tag);
        var now = store.get(k) !== '1';
        store.set(k, now ? '1' : '0');
        card.classList.toggle('done', now);
        if (now) confettiRain();
        updateProgress();
        renderTodayCard();
        updateTodayStates();
        if (now && !wasDone && dayDone(tag)) dayFireworks();
      });
```

- [ ] **Step 2: `dayFireworks()` ergänzen**

Im selben Scope wie die Engine (Task 1) einfügen — fünf zeitversetzte Raketen quer über die Seite, plus Toast (Toast auch bei reduzierter Bewegung, da rein statisch):

```javascript
  function dayFireworks(){
    celebrateToast();
    if (REDUCE) return;
    var shots = [[0.25,0.30],[0.55,0.20],[0.78,0.35],[0.40,0.18],[0.66,0.40]];
    shots.forEach(function(s, k){
      setTimeout(function(){ firework(innerWidth*s[0], innerHeight*s[1]); }, k*340);
    });
  }
```

- [ ] **Step 3: Im Browser verifizieren**

Neu laden. Einen Tag mit mehreren Terminen wählen, alle bis auf einen abhaken (jeweils Konfetti, kein Feuerwerk). Den letzten abhaken → **Feuerwerk + „🎉 Tag geschafft!"-Toast**. Einen Termin wieder aufheben, dann wieder abhaken → Feuerwerk feuert erneut (erneuter Übergang). Bei bereits komplettem Tag einen anderen Termin an-/abhaken löst **kein** zusätzliches Feuerwerk aus, solange kein neuer Übergang offen→komplett passiert.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Belohnungs-Effekte: Feuerwerk + Toast beim Tagesabschluss"
```

---

### Task 4: Reduced-Motion-Statt-Moment beim Abhaken

**Files:**
- Modify: `index.html` — CSS (kurze Aufleucht-Animation) + Karten-Handler.

**Interfaces:**
- Consumes: `REDUCE`-Flag aus Task 1; `card` aus dem Handler.
- Produces: CSS-Klasse `.appt.flash` + `@keyframes tp-flash`.

Hintergrund: Bei `prefers-reduced-motion: reduce` geben `confettiRain()`/`firework()` sofort zurück (Task 1). Damit der Moment trotzdem spürbar ist, leuchtet die abgehakte Karte kurz auf. Der Toast beim Tagesabschluss bleibt (ist statisch).

- [ ] **Step 1: CSS für das Aufleuchten ergänzen**

Im `<style>`-Block einfügen:

```css
  @keyframes tp-flash { 0% { box-shadow: 0 0 0 0 rgba(110,168,254,0); } 30% { box-shadow: 0 0 0 3px rgba(110,168,254,.55); } 100% { box-shadow: 0 0 0 0 rgba(110,168,254,0); } }
  .appt.flash { animation: tp-flash .6s ease-out; }
```

- [ ] **Step 2: Aufleuchten im Handler auslösen (nur bei REDUCE)**

Die Konfetti-Zeile aus Task 2 (`if (now) confettiRain();`) ersetzen durch:

```javascript
        if (now){
          if (REDUCE){ card.classList.add('flash'); setTimeout(function(){ card.classList.remove('flash'); }, 650); }
          else confettiRain();
        }
```

- [ ] **Step 3: Im Browser verifizieren**

In den OS-Einstellungen „Bewegung reduzieren" aktivieren (macOS: Systemeinstellungen → Bedienungshilfen → Anzeige → Bewegung reduzieren; oder DevTools → Rendering → „Emulate prefers-reduced-motion: reduce"). Seite neu laden. Termin abhaken → **kein** Konfetti, stattdessen kurzes Aufleuchten der Karte. Tag komplett abhaken → **kein** Feuerwerk, nur der Toast. Danach „Bewegung reduzieren" wieder aus → volle Effekte zurück.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Belohnungs-Effekte: dezenter Statt-Moment bei reduzierter Bewegung"
```

---

### Task 5: Dokumentation aktualisieren

**Files:**
- Modify: `CLAUDE.md`, `README.md`.

**Interfaces:**
- Consumes: nichts.
- Produces: nichts.

- [ ] **Step 1: README ergänzen**

Unter „## Funktionen" in `README.md` einen Punkt ergänzen:

```markdown
- Belohnungs-Effekte: Konfetti beim Abhaken eines Termins, Feuerwerk + „Tag geschafft"-
  Einblendung bei komplettem Tag (respektiert „Bewegung reduzieren")
```

- [ ] **Step 2: CLAUDE.md ergänzen**

In `CLAUDE.md` unter „## Funktionen" einen Punkt ergänzen und einen kurzen Erklär-Absatz, wo die Effekte sitzen:

```markdown
- Belohnungs-Effekte: Konfetti beim Abhaken, Feuerwerk + Toast bei komplettem Tag.
```

Sowie als neuer Abschnitt (z. B. nach „Heute-Karte …"):

```markdown
## Belohnungs-Effekte (Konfetti + Feuerwerk)
Canvas-Overlay `#fx` + Partikel-Engine inline in `index.html` (keine Library). Konfetti
(`confettiRain()`) feuert im Karten-Klick-Handler beim Abhaken; das Feuerwerk
(`dayFireworks()`) bei Übergang offen→komplett (Vorher/Nachher-Vergleich von `dayDone()`).
`prefers-reduced-motion: reduce` schaltet die Partikel ab (`REDUCE`-Flag) und ersetzt sie
durch ein kurzes Aufleuchten der Karte; der „Tag geschafft"-Toast bleibt. Intensität/Farben
über `FXCOL`, `confettiRain`/`firework`-Parameter; Partikel-Deckel `MAX_PARTS`.
```

- [ ] **Step 3: Verifizieren**

`README.md` und `CLAUDE.md` lesen — die neuen Stellen stehen drin, Markdown sauber, keine Privatdaten.

- [ ] **Step 4: Commit**

```bash
git add README.md CLAUDE.md
git commit -m "Doku: Belohnungs-Effekte beschrieben"
```

---

## Self-Review (vom Plan-Autor)

**Spec coverage:**
- Canvas-Overlay + selbst-beendender Loop → Task 1. ✓
- Konfetti beim Abhaken, nur bei „on" → Task 2. ✓
- Feuerwerk + Toast genau am Übergang offen→komplett → Task 3. ✓
- `prefers-reduced-motion` (keine Partikel, Statt-Moment, Toast bleibt) → Task 1 (Flag/Guards) + Task 4 (Statt-Moment). ✓
- Partikel-Obergrenze → Task 1 (`MAX_PARTS`, in `confettiRain`/`firework` gedeckelt). ✓
- Effekte an jedem Tag → ergibt sich aus dem Handler (kein Heute-Filter). ✓
- Keine Vibration/Sound/Abschalt-Option → nicht eingeplant (gewollt). ✓
- Doku-Hinweis → Task 5. ✓

**Placeholder-Scan:** keine TBD/TODO/„später"; jeder Code-Schritt zeigt vollständigen Code. ✓

**Typ-/Namens-Konsistenz:** `confettiRain()`, `firework(x,y)`, `celebrateToast()`, `dayFireworks()`, `REDUCE`, `MAX_PARTS`, `parts`, `run()` durchgängig gleich benannt zwischen Tasks 1–4. `dayDone(tag)` ist bestehende Funktion (`index.html:407`). ✓
