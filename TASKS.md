# Feld – TASKS

## Session 2026-10-07: Erstellung der App (von Grund auf)

Ausgangslage: `_App Konzept/KONZEPT.md` + `_App Konzept/feld-daten.json` (Erstbefüllung:
134 Akteure, 83 Veranstaltungen, 24 Ideen) lagen bereits vor. Daraus wurde die App gebaut.

### Gemacht
- Ordnerstruktur `programm/` (App) + `feld-data/` (Daten) angelegt — **User-Entscheidung:**
  Ordnernamen klein geschrieben (`programm`, nicht `Programm`), da der User sie so selbst
  angelegt hat (ihm fehlten zunächst ACL-Schreibrechte auf `app feld/`, siehe unten).
- `programm/index.html`: Single-File-App, dunkles Design (Fundraising/Zeiterfassung-Familie,
  Akzentfarbe Blau statt Grün, um sich optisch leicht zu unterscheiden), kein Framework,
  kein Build-Schritt.
  - Datenschicht: `Store.list/get/put/remove` (Tombstone-Löschung), referenzielle Integrität
    beim Löschen (Warnung mit Liste der Verweise, zweistufiger Löschen-Button, Verweise werden
    bei Bestätigung bereinigt statt verwaist zu bleiben).
  - Alle drei Ansichten (Verzeichnis/Agenda/Ideen) inkl. Filter, A-Z-Gruppierung,
    Anstehend/Vergangen mit „Heute"-Trennlinie, Ideen-Karten.
  - Suche über alle drei Typen, diakritika-unabhängig (NFD-Normalisierung), UND-verknüpft.
  - Detail-Dialog (natives `<dialog>`) mit Rückverweisen, die innerhalb des Dialogs navigieren
    (Stack mit Zurück-Button), Textbadges für „zusammengearbeitet"/„eigene".
  - Formulare für alle drei Typen, Beteiligte/Akteure-Zeilen mit Datalist-Namensvorschlägen,
    unbekannte Namen werden beim Speichern automatisch als neue Akteure angelegt
    (Person, bzw. Institution bei „Gehört zu").
  - Themenvokabular inline erweiterbar (im Formular UND in den Einstellungen), persistiert.
  - Export/Import als JSON, inkl. herunterladbarer Vorlage mit Formaterklärung.
  - GitHub-Sync (Contents API, PAT in Einstellungen) — **noch nie gegen ein echtes Repo
    getestet**, nur der Code-Pfad (siehe „Offen" unten).
  - Tastatur: `/` fokussiert Suche, Pfeiltasten wechseln Reiter, Esc schliesst Dialoge
    (nativ). ARIA-Tabs, `aria-pressed` auf Filtern, Live-Region für Trefferzahl.
- Icons selbst gestaltet (kein Logo vom User vorgegeben): drei verbundene Knoten als Sinnbild
  für „drei Objekttypen, ein Netz" (Leitsatz aus KONZEPT.md). Mit Pillow gerendert (kein
  `rsvg-convert`/`cairosvg` verfügbar), `icon.svg` als bearbeitbare Quelle liegt bei.
- `feld-data/feld-daten.json` aus der Erstbefüllung übernommen, ergänzt um `themenVokabular`
  (die 12 Kernthemen aus KONZEPT 3.4) **plus `Fussverkehr` und `Klang`** — diese zwei Themen
  wurden in den Akteur-Daten bereits verwendet, standen aber nicht im Kern-Vokabular. Beim
  Erstbefüllen ergänzt, damit das Vokabular von Anfang an vollständig ist.
- Getestet (Playwright headless über echtes System-Chrome, `playwright-core` +
  `executablePath` auf `/Applications/Google Chrome.app/...`, da die Chrome-Erweiterung diese
  Session nicht verbunden war): Laden ohne Fehler, Import der echten Daten (Zähler exakt
  134/83/24), alle drei Ansichten inkl. Filter, Suche (inkl. Diakritika), Detail-Dialog +
  Rückverweis-Navigation mit Zurück-Button, Neuanlage, Bearbeiten, Löschen mit
  Referenz-Warnung (inkl. Verifikation, dass Verweise tatsächlich bereinigt werden — per
  direkter `DATA`-Inspektion, nicht nur visuell), Themenvokabular-Erweiterung inline,
  Export-Download, Tastaturkürzel (`/`, Pfeiltasten), Mobile-Viewport (390px).
- Ein echter Bug beim Testen gefunden und gefixt: CSS-Regel `dialog { display: flex; }` gab
  allen `<dialog>`-Elementen permanent `display:flex`, auch ohne `open`-Attribut — die
  Browser-UA-Regel `display:none` für geschlossene Dialoge wurde dadurch überschrieben, alle
  drei Dialoge waren beim Laden sichtbar (unmoduliert, ohne Backdrop, in-flow). Fix: auf
  `dialog[open] { display: flex; }` umgestellt.
- Kleine Mobile-Korrektur: Header-Zeile wrappte bei sehr schmalen Viewports (Zahnrad-Icon
  rutschte in eine eigene Zeile) — `flex-wrap: nowrap` + kleinere `flex-basis` fürs Suchfeld
  unter 560px behoben.

### Offen / nächster Schritt
1. **Git-Commit fehlt noch in beiden Repos** (`programm/`, `feld-data/`) — beide sind
   `git init` + `git add -A` (alle Dateien gestaged), aber **nicht committet**. Grund: keine
   Git-Identität (`user.name`/`user.email`) konfiguriert, weder global noch lokal; User wollte
   das selbst übernehmen („das Login mache ich, muss dich nicht kümmern"). Nächster Schritt:
   User committet selbst (oder nennt Name/E-Mail für einen Commit durch Claude).
2. **Kein GitHub-Repo existiert bisher** — User-Entscheidung dieser Session: nur lokal
   vorbereiten, Repo-Erstellung/Push für eine spätere Session. Repo-Namen (vom User
   festgelegt): `feld-programm` (Programm) / `feld-data` (Daten).
3. **GitHub-Sync nie gegen ein echtes Repo getestet** (kein Repo vorhanden). Sobald die Repos
   existieren: in den Einstellungen verbinden (Owner/Repo/Pfad/PAT), „Verbinden &
   synchronisieren" testen, prüfen ob `feld-daten.json` korrekt hochgeladen wird.
4. **Datenübernahme**: Die App wurde bisher nur mit der Datei aus `_App Konzept/feld-daten.json`
   importgetestet (lokal im Browser, nicht persistent). Die „echte" Erstbefüllung passiert erst,
   wenn der User selbst entweder importiert oder GitHub-Sync gegen `feld-data/feld-daten.json`
   herstellt.
5. **Rechteproblem beim Ordner `app feld/` selbst** (nicht die Unterordner): Der User (tulip)
   hatte zu Sessionbeginn keine Schreibrechte auf `/Users/Shared/Claude/projekte/app feld/`
   (im Unterschied zu den Geschwisterordnern wie `app fundraising`, die eine ACL für tulip
   haben). Der User hat `programm/` und `feld-data/` daraufhin selbst angelegt (daher
   Kleinschreibung). Diese Notiz liegt deshalb in `programm/TASKS.md` statt im Projekt-Wurzel-
   ordner — dort hat Claude weiterhin keine Schreibrechte. Falls das stört: ACL wie bei den
   anderen App-Ordnern ergänzen (`chmod +a "user:tulip allow ..." "app feld"`).
6. Barrierefreiheit (WCAG 2.2 AA) wurde nur strukturell umgesetzt (ARIA-Tabs, aria-pressed,
   native Dialoge, Live-Region, prefers-reduced-motion) — **nicht** mit echtem Screenreader
   (VoiceOver/NVDA) getestet, wie im Konzept unter 5.6 als offener Prüfpunkt vermerkt.
7. Gestaltung (KONZEPT 5.5) war offen — User-Entscheidung dieser Session: dunkles Design wie
   die anderen Apps, Akzentfarbe Blau.

## Session 2026-10-07 (Folgesession): Sync-Fix, Kurznotiz, Ideen-Sortierung & Ordner

### Gemacht
- **Sync-Punkt-Bug gefixt**: `setSyncStatus()` setzte die CSS-Klasse `idle` (verbunden &
  synchronisiert), gestylt war aber nur `.sync-dot.synced` — der Punkt blieb grau statt grün.
  Fix: CSS-Regel in `.sync-dot.idle` umbenannt (index.html, CSS-Bereich oben). Mit Playwright
  verifiziert (`getComputedStyle` → `rgb(74,222,128)`).
- **Akteure**: neues Feld `kurznotiz` (max. 100 Zeichen, Konstante `KURZNOTIZ_MAXLEN`) zusätzlich
  zum bestehenden `notiz` (unbegrenzt). `kurznotiz` erscheint **vollständig, ungekürzt** direkt in
  der Verzeichnis-Zeile (kein Klick nötig); ist `kurznotiz` leer, wird ersatzweise `notiz`
  zweizeilig gekürzt angezeigt (Rückwärtskompatibilität für die 134 Alteinträge ohne
  `kurznotiz`). Formularfeld mit Live-Zeichenzähler. In Suche, Template/Export integriert.
- **Ideen**: grundlegend überarbeitet, User-Wunsch „sortieren können" umgesetzt:
  - Von Karten-Raster (`.cards`) auf einfache Liste (`.row`, wie Verzeichnis/Agenda) umgestellt.
  - Sortier-Chips: Neu (Standard) / Alt / A-Z / Manuell, Zustand in `localStorage` persistiert.
  - Neuer Datentyp `ideenOrdner` (eigene Collection in `DATA`, gleiches Tombstone-Lösch-Muster
    wie akteure/veranstaltungen/ideen) — Ideen lassen sich Ordnern zuordnen (Feld `ordner` auf
    der Idee). Anlegen über „+ Ordner" im Toolbar (Ideen-Reiter) oder inline im Idee-Formular;
    Umbenennen per Klick auf ✎ (inline editierbar); Löschen über den bestehenden
    zweistufigen `attachDeleteButton`-Mechanismus (Ideen verlieren dabei nur die
    Ordner-Zuordnung, werden nicht gelöscht — referenzielle Integrität wie bei
    Akteuren/Veranstaltungen).
  - Manuelle Sortierung: Feld `reihenfolge` (Zahl) auf der Idee. Im Modus „Manuell" erscheinen
    pro Idee ▲/▼-Buttons (tastaturfreundliche Alternative zu Drag&Drop); zusätzlich echtes
    HTML5-Drag&Drop: Ziehen zwischen Zeilen (nur im Modus „Manuell") positioniert fein,
    Ziehen auf einen Ordner-Header (**in jedem Sortiermodus**) verschiebt die Idee in diesen
    Ordner. Beides nummeriert die betroffene Zielgruppe komplett neu durch (Schrittweite 10).
  - Kleiner generischer Fix in `attachDeleteButton`: `btn.closest('.dialog-foot, form')` konnte
    `null` liefern (Ordner-Löschen-Button sitzt in keinem von beidem) → Fallback auf `btn` selbst,
    sonst wäre beim Löschen eines referenzierten Ordners ein JS-Fehler geflogen.
- **Getestet** mit Playwright headless (`playwright-core`, System-Chrome, selbes Rezept wie
  Erstsession, diesmal via `npm install --no-save playwright-core` im Scratchpad): Kurznotiz
  anlegen + Zähler + Anzeige in der Zeile, Ordner anlegen/umbenennen/löschen (inkl. Warnbox bei
  Verweisen, Ideen bleiben erhalten), Sortier-Umschaltung, Drag&Drop Zeile→Zeile (Manuell-Modus)
  und Zeile→Ordner-Header (alle Modi), Sync-Punkt-Farbe. Keine Konsolenfehler.
- Datenschicht (`loadLocal`/`saveLocal`/`currentPayload`/`mergeIncomingPayload`/`uid`/
  `Store.put`/`referencingFor`/`stripReferencesTo`) durchgängig um `ideenOrdner` ergänzt, damit
  GitHub-Sync und Tombstone-Löschung auch für Ordner funktionieren.

### Offen / nächster Schritt
1. Sync-Bug (grauer Punkt) war eigentlich ein zweites, am Session-Anfang vermutetes Problem
   (Login-Unterschied tulip/lw) — das war laut User nur ein Tippfehler beim Login, der grau
   bleibende Punkt danach war der eigentliche (jetzt gefixte) CSS-Bug.
2. `KURZNOTIZ_MAXLEN = 100` ist eine Annahme (User hat keine genaue Zahl genannt) — bei Bedarf
   anpassen (eine Konstante, index.html, Suche `KURZNOTIZ_MAXLEN`).
3. User commitet selbst und kopiert den Code-Stand manuell zu `lw` hinüber (eigener Workflow,
   kein GitHub-Push von hier aus nötig/gewünscht) — **diese Session hat nichts committet**,
   nur Dateien geändert.
4. Bestehende offene Punkte aus der Erstsession (s. oben, Nr. 1–6) weiterhin unverändert offen,
   insbesondere: noch kein Git-Commit, kein GitHub-Repo, Sync nie gegen echtes Repo getestet.
