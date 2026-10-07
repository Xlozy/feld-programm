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
