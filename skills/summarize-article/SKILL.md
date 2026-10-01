---
name: summarize-article
description: Use only when explicitly invoked via /summarize-article. Never auto-trigger.
---

# Artikel zusammenfassen

Verarbeite **{$ARGUMENTS}** — je nach Argument-Typ im passenden Modus.

## Modus-Erkennung

| Argument | Modus | Aktion |
|----------|-------|--------|
| URL (`http://…` / `https://…`) | **Neu erstellen** | Artikel abrufen, zusammenfassen und mit Originaltext speichern |
| Dateipfad (`.md`) | **Aktualisieren (Einzeldatei)** | Zusammenfassung und Originaltext neu generieren |
| Verzeichnispfad | **Aktualisieren (Batch)** | Alle MD-Dateien im Verzeichnis aktualisieren |

## Vorgehen: Neu erstellen (URL)

1. **Abrufen** des Artikels (WebFetch, bei 403/401 cURL-Fallback)
2. **Originaltext sichern:** Den abgerufenen Artikeltext (bereinigt um Navigation, Werbung, Footer etc.) für den `<details>`-Block aufbewahren
3. **Duplikat-Check:** Prüfen ob unter `/Users/valgard/Documents/!AI/article_summaries/` bereits eine Datei für diesen Artikel existiert → falls ja, den User fragen: überschreiben, abbrechen oder neuen Slug wählen?
4. **Schreiben** der MD-Datei im unten beschriebenen Format (inkl. Originaltext im `<details>`-Block)
5. **Speichern** nach `/Users/valgard/Documents/!AI/article_summaries/{dateiname}.md`

**Bei Zugriffsfehler** (Paywall, 403 + cURL-Fallback schlägt fehl): User informieren und abbrechen. Keine Teildatei auf Basis unvollständiger Inhalte erstellen.

## Vorgehen: Aktualisieren (Einzeldatei)

1. **Datei lesen** — bestehende Metadaten (Autor, Quelle, Datum) und Dateiname merken
2. **Quell-URL extrahieren** aus den Metadaten (`- **Quelle:** <URL>`)
3. **Artikel abrufen** (WebFetch, bei 403/401 cURL-Fallback)
4. **Datei komplett neu schreiben** — Zusammenfassung und Originaltext auf Basis des aktuellen Artikeltexts neu generieren, im selben Format wie bei "Neu erstellen"
5. **Dateiname beibehalten** — den bestehenden Dateinamen nicht ändern

**Bei Zugriffsfehler:** User informieren, Datei unverändert lassen.

## Vorgehen: Aktualisieren (Batch)

1. **Verzeichnis scannen** — alle `.md`-Dateien auflisten
2. **User informieren** — Anzahl der zu aktualisierenden Dateien anzeigen, Bestätigung einholen
3. **Nacheinander aktualisieren** — für jede Datei den Einzeldatei-Ablauf (s.o.) durchführen
4. **Übersicht** — am Ende: wie viele aktualisiert, wie viele übersprungen (Zugriffsfehler etc.)

## Dateiname

Slug-Formel: **Kernbegriffe aus dem Titel + Quelle/Autor + Jahr**

- Lowercase, Bindestriche statt Leerzeichen
- Stoppwörter weglassen (a, the, my, going, into, of, …)
- Kein Datumspräfix, aber Veröffentlichungsjahr als Suffix (bei unbekanntem Jahr: weglassen)
- Beispiele:
  - "Spec-Driven Development" (ThoughtWorks, 2025) → `spec-driven-development-thoughtworks-2025.md`
  - "My LLM Coding Workflow Going into 2026" (Addy Osmani) → `llm-coding-workflow-addy-osmani-2026.md`
  - "No Coding Before 10am" (Michael Bloch, 2026) → `no-coding-before-10am-bloch-2026.md`

## Format der MD-Datei

~~~markdown
# Originaltitel des Artikels

- **Autor:** Name des Autors
- **Quelle:** <URL>
- **Datum:** Veröffentlichungsdatum (z.B. "11. Februar 2026")

---

## Kernthese

Ausformulierte Kernaussage des Artikels in 1-3 Sätzen. Reiner Fließtext, keine Inline-Formatierung.

---

## {Thematischer Abschnitt 1}

[Hauptthemen in eigenen ##-Abschnitten zusammenfassen. ###-Unterabschnitte, Bullet-Listen und Tabellen wo sinnvoll.]

## {Thematischer Abschnitt 2}

[Weitere Abschnitte je nach Artikelumfang und -struktur.]

---

## Fazit

Zusammenfassendes Fazit des Autors oder eigene Einordnung.

---

<details>
<summary>Originaltext</summary>

[Vollständiger, bereinigter Artikeltext — ohne Navigation, Werbung, Footer, Cookie-Banner etc.]

</details>
~~~

### Originaltext-Block

- Der `<details>`-Block steht **nach dem letzten `---`-Trenner**, ganz am Ende der Datei
- **Bereinigung:** Nur den eigentlichen Artikeltext speichern — Navigation, Werbung, Sidebar, Footer, Cookie-Banner, Social-Media-Buttons etc. entfernen
- **Formatierung:** Den Text als sauberes Markdown aufbereiten (Überschriften, Absätze, Listen beibehalten), nicht als rohes HTML
- **Sprache:** Originalsprache des Artikels beibehalten (nicht übersetzen)
- **Typora-Kompatibilität:** Innerhalb des `<details>`-Blocks keine doppelten Leerzeilen verwenden — Typora splittet den Block sonst in separate HTML-Blöcke

### Verbindliche Format-Details

- **H1-Titel:** Immer der **Originaltitel** des Artikels (nicht übersetzen)
- **Überschriften `## Kernthese` und `## Fazit`:** Exakt so verwenden — keine Varianten ("Kernidee", "Fazit des Autors")
- **Metadaten-Format:** Ungeordnete Liste mit `- **Feld:** Wert` — keine Blockquotes, keine andere Formatierung
- **Metadaten-Reihenfolge:** Autor → Quelle → Datum
- **Quelle:** Nackte URL in spitzen Klammern `<https://...>` — kein benannter Markdown-Link
- **`---`-Trennlinien:** Nur zwischen den fünf Hauptblöcken: Metadaten / Kernthese / Hauptteil / Fazit / Originaltext — **nicht** zwischen einzelnen thematischen Abschnitten
- **Unterüberschriften (`###`):** Erlaubt innerhalb thematischer `##`-Abschnitte für Untergliederung
- **Umlaute:** UTF-8 verwenden (ä, ö, ü, ß) — keine Umschreibungen (ae, oe, ue)

### Fehlende Metadaten

| Feld | Fallback |
|------|----------|
| Autor | `Unbekannt` — nicht die Domain oder Organisation einsetzen |
| Datum | Aus URL oder Meta-Tags ableiten, Format: "März 2025 (geschätzt)". Wenn nichts ableitbar: `Unbekannt` |

## Stilregeln

- **Sprache:** Deutsch, Fachbegriffe dürfen englisch bleiben
- **Ton:** Sachlich, nah am Original, aber eigenständig formuliert
- **Abschnitte** richten sich nach dem Artikelinhalt — kein festes Schema
- **Tabellen** für strukturierte Vergleiche oder Auflistungen
- **Tiefe:** Nachvollziehbar und ausführlich, nicht nur oberflächliche Bullet-Points. Richtwert: 20–30 % der Originallänge
- Keine Emojis
