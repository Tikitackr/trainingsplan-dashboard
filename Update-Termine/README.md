# Update-Termine

**Hier legst du einen neuen Therapieplan-Scan rein** (Foto/PDF/PNG), wenn sich
die Termine ändern.

Ablauf danach:
1. Du legst die Datei hier ab und startest Claude (oder sagst „neuer Plan").
2. Claude liest den Scan, ignoriert alle Privatdaten (Name, Diagnose, Zimmer …),
   und baut die Termine neu in `plan.json` (und im `PLAN_DEFAULT`-Block der
   `index.html`).
3. Claude committet + pusht. GitHub Pages baut neu, das iPhone zeigt die neuen
   Termine beim nächsten Öffnen automatisch (kein Cache löschen nötig).
4. Der abgearbeitete Scan wandert nach `../Termine/` (Archiv).

**Wichtig:** Die Scans hier enthalten Privatdaten und sind per `.gitignore` vom
öffentlichen Repo ausgeschlossen. Nur diese README liegt im Repo.

Hinweis: Claude läuft nicht im Hintergrund. „Automatisch" heißt: Datei ablegen
und eine Session starten — es springt nicht von allein an.
