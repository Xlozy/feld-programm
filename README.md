# Feld

Persönliches Arbeitswerkzeug: Verzeichnis für Gehkultur und Spaziergangswissenschaft.
Überblickt das Feld, in dem gearbeitet wird — Akteure, Veranstaltungen und Ideen als ein Netz.

Konzept und Datenmodell: siehe `../_App Konzept/KONZEPT.md`.

## Technik

- Eine einzelne HTML-Datei (`index.html`), kein Build-Schritt, kein Framework.
- Daten liegen als JSON in einem separaten Repository (`feld-data`), eingebunden über die
  GitHub-Contents-API mit einem Personal Access Token (Einstellungen → GitHub-Sync).
- Lokale Kopie der Daten liegt zusätzlich in `localStorage`, die App funktioniert auch offline;
  Sync gleicht pro Eintrag anhand des neueren `modified`-Zeitstempels ab.
- Export/Import als JSON-Datei (Einstellungen), inkl. herunterladbarer Vorlage, die das
  Importformat erklärt.

## Verwenden

`index.html` lokal im Browser öffnen (funktioniert auch direkt per `file://`, z. B. in Safari
„Zum Dock hinzufügen"). Beim ersten Start ist die App leer — entweder über Einstellungen →
GitHub-Sync mit dem Daten-Repo verbinden, oder eine JSON-Datei importieren.

## Icons

`icon.svg` ist die bearbeitbare Quelle (drei verbundene Knoten = die drei Objekttypen als ein
Netz). PNG-Grössen und `favicon.ico` sind daraus exportiert — bei Änderungen am Motiv müssen sie
neu gerendert werden (z. B. mit Pillow, Rezept siehe Session-Notiz).
