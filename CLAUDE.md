# CLAUDE.md – Bottleneck Vault (Repo: Aktien)

## Was das hier ist
Persönliche Aktien-Ideen- und Portfolio-Notizapp. Eine einzige Datei, `index.html`, kein Build-Schritt, keine Abhängigkeiten. Wird über GitHub Pages ausgeliefert (`wieskoenig-ops.github.io/Aktien`) und auf dem iPad als Home-Bildschirm-App genutzt.

Zweck: schnelle Notizen zu Aktien (Idee, Thesis, Links), dazu Bestand und Watchlist im Blick. Kein Trading-Tool, keine Kurs-Feeds.

## Harte Regeln
1. **`STORAGE_KEY` nie ändern** (aktuell `bottleneck-vault-v2`). Alle Daten liegen im `localStorage` des Geräts. Ein neuer Key lässt alle Notizen verschwinden.
2. **Die URL nie ändern** (kein Umbenennen des Repos, keine Custom Domain). `localStorage` hängt am Origin.
3. **`INITIAL` wird nur beim allerersten Start geschrieben**, wenn `localStorage` leer ist. Änderungen an `INITIAL` erreichen bestehende Geräte nicht. Neue Positionen für bestehende Nutzer gehen nur über Export/Import oder manuell über „+ Neu".
4. **Eine Datei bleibt eine Datei.** Kein Framework, kein Bundler, keine externen CDNs. CSS und JS bleiben inline in `index.html`.
5. **UI-Sprache ist Deutsch.**
6. **Keine Kurs-APIs, keine Netzwerkaufrufe.** Kurse und Zahlen werden von Hand eingetragen.
7. **Das Repo ist öffentlich.** Keine persönlichen Daten über das Nötige hinaus in Dateien, Commits oder diese Anleitung schreiben.

## Design-Prinzipien
- **iPad zuerst** (Hoch- und Querformat), danach Handy, Desktop nur nebenbei. Touch-Ziele groß, Bottom-Navigation.
- **Notizen sind der wichtigste Inhalt.** In der Liste sieht man eine 2-Zeilen-Vorschau, in der Detailansicht steht das Notizfeld ganz oben. Kennzahlen sind eingeklappt („Kennzahlen & Details").
- **Schnellerfassung:** „+ Neu" zeigt zuerst nur Ticker, Name, Status (Beobachten / Bestand / Watchlist) und ein großes Notizfeld. Alles andere ist optional und eingeklappt.
- Notizen unterstützen minimal Markdown: `[Text](URL)`, `**fett**`, `*kursiv*`. Alles läuft durch `h()` (HTML-Escaping), bevor ersetzt wird. Das nicht aufweichen.
- Dunkles Theme, CSS-Variablen in `:root`.

## Aufbau von index.html
- `INITIAL`: Array mit allen Startpositionen
- Zustand: `data`, `currentView`, Filter-Variablen, `selected`, `editData`, `editMode`
- `load()`, `save()`, `nav()`, `render()`
- `renderDash()`, `renderList()`, `renderDetail()`, `renderNew()`, `renderMarkdown()`
- `bindEvents()` – **eine einzige Funktion**, wird nach jedem `render()` aufgerufen
- `renderBackup()`, `exportFile()`, `exportCopy()`, `importJSON()`
- Views: `dash`, `list`, `watch`, `detail`, `new`, `backup`

## Datenmodell einer Position
```
id, ticker, name, sektor, thema, region, währung,
status            // Bestand | Beobachten | Watchlist | Verkauft
conviction        // hoch | moderat-bullish | neutral | leicht-bullish | neutral-spekulativ
bottleneck        // true/false
bottleneck_typ, stück, positionswert, einstiegskurs, perf_pct, perf_eur,
kgv_forward, kgv_median_5j, dividende, bewertung,
nächstes_earnings (YYYY-MM-DD), catalyst, einstieg_ziel,
scalable_thema    // Bucket, siehe unten
tags[], notizen, letzte_aktualisierung (YYYY-MM-DD)
```
Fast alle Felder sind optional und dürfen `null` oder leer sein. Code muss damit umgehen können.

Buckets (`scalable_thema`): Kern, Robotik, Deutschland, Tech International, Gesundheit, Rohstoffe, China, Europa Infrastruktur, Bottleneck Watchlist, Deutsche Bank.

## Arbeitsweise
- **Zuerst die Datei lesen, dann gezielt editieren.** Keine String-Replace-Patches mit großen Blöcken aus dem Gedächtnis. Das hat früher doppelte Deklarationen und kaputte Handler erzeugt (z. B. `newDetToggle` doppelt deklariert, dadurch weiße Seite).
- **Nach jeder Änderung Syntax prüfen:** das `<script>` in eine Temp-Datei extrahieren und mit `node --check` laufen lassen. Außerdem prüfen, dass `bindEvents` genau einmal vorkommt.
- Bestehende Funktionen ändern, nicht daneben neue Varianten bauen.
- Kleine, getrennte Commits mit verständlicher Nachricht. Push auf `main` löst das Pages-Deployment aus.
- Layout, CSS und Funktionen ändern ist unkritisch. Alles, was `STORAGE_KEY`, das Datenmodell oder die Speicherlogik berührt, vorher ansprechen.
- Datenmodell nur **additiv** erweitern. Neue Felder müssen bei alten gespeicherten Datensätzen fehlen dürfen (Defaults im Code).

## Offene Punkte
1. ~~Export/Import als JSON~~ – erledigt: View `backup` (Header-Button „⇅ Backup“). Export als Datei (Share-Sheet auf dem iPad) oder Zwischenablage; Format `{app, version, exportiert, anzahl, positionen:[…]}`. Import akzeptiert dieses Format oder ein nacktes Array und **ersetzt** alle Daten (mit Rückfrage).
2. Neue Positionen als JSON-Schnipsel importierbar machen (Merge nach `id`, bestehende Notizen nicht überschreiben).
3. Suche auch in Notizen und Tags.
4. Sortieroptionen in der Liste (Wert, Performance, nächstes Earnings).

## Inhaltlich (nur zum Verständnis)
Titel werden nach einem Bottleneck-Framework eingeordnet: Lieferketten-Engpässe mit Monopol- oder Oligopolcharakter, im Gegensatz zu reiner Nachfrage-Story. Das Flag `bottleneck` und `bottleneck_typ` bilden das ab.
