---
name: competitor-research
description: Analysiert eine oder mehrere Konkurrenz-Domains und identifiziert deren bestperformende Blog/Ratgeber-Inhalte als Basis für eigene Content-Erstellung. Nutzt Ahrefs MCP für Traffic-, Keyword- und Backlink-Daten.
---

# Modul 1: Competitor Research

## Wann diesen Skill verwenden
Wenn der User eine Konkurrenz-Domain analysieren möchte, um deren erfolgreichste Blog-Inhalte zu identifizieren. Dies ist typischerweise der erste Schritt beim Erstellen eines neuen Blogartikels.

## Inputs
- **Pflicht**: Mindestens eine Konkurrenz-Domain (z.B. `garten-beispiel-eins.example`) – oder aus `competitors[]` der Site-Config
- **Pflicht**: Site-Config (`/config/sites/[domain].json`) – für Relevanz-Bewertung und Ländercode
- **Optional**: Ziel-Keyword oder Themenbereich (z.B. "gartenmöbel", "sonnenschirm", "lounge")
- **Optional**: Anzahl Top-Ergebnisse (Standard: 10)

## Output
Eine strukturierte Datei `competitor-research-results.json` mit:
- Domain-Übersicht (Traffic, Keywords, Domain Rating)
- Top Blog/Ratgeber-Seiten sortiert nach Traffic
- Keywords pro Seite
- Referring Domains pro Seite
- Empfehlung welche Themen das größte Potenzial haben

---

## Ablauf

### Schritt 1: Domain-Übersicht holen

Verwende `ahref:site-explorer-metrics` um einen Überblick über die Konkurrenz zu bekommen:

```
Tool: ahref:site-explorer-metrics
Parameter:
  target: [konkurrenz-domain]
  date: [aktuelles Datum, Format YYYY-MM-DD]
  country: de
  mode: subdomains
```

Notiere:
- `org_keywords` – Gesamtanzahl organischer Keywords
- `org_traffic` – Geschätzter monatlicher organischer Traffic
- `org_cost` – Traffic-Wert in USD-Cents

### Schritt 2: Top-Seiten nach Traffic identifizieren

Verwende `ahref:site-explorer-top-pages` um die trafficstärksten Seiten zu finden:

```
Tool: ahref:site-explorer-top-pages
Parameter:
  target: [konkurrenz-domain]
  date: [aktuelles Datum, Format YYYY-MM-DD]
  country: DE
  mode: subdomains
  limit: 30
  select: url,top_keyword,top_keyword_volume,sum_traffic,keywords,referring_domains
```

**Wichtig**: Verwende NICHT `order_by` bei diesem Endpoint – die Ergebnisse kommen standardmäßig nach Traffic sortiert.

### Schritt 3: Blog/Ratgeber-Content filtern

Aus den Top-Seiten die Content-Seiten identifizieren. Typische Subfolder:
- `/ratgeber/`
- `/blog/`
- `/magazin/`
- `/wissen/`
- `/tipps/`
- `/artikel/`
- `/guide/`

**Automatische Erkennung**: Scanne die URLs der Top 30 Seiten. Identifiziere wiederkehrende Pfad-Segmente, die auf redaktionellen Content hinweisen. Ignoriere:
- Reine Produktseiten (`/produkt/`, `/shop/`, `/p/`)
- Kategorie-Seiten (kurze URLs ohne tiefe Pfadstruktur)
- Rechtliche Seiten (`/impressum`, `/datenschutz`, `/agb`)

Falls ein Content-Subfolder identifiziert wird, mache einen zweiten gezielten API-Call:

```
Tool: ahref:site-explorer-top-pages
Parameter:
  target: [konkurrenz-domain]
  date: [aktuelles Datum]
  country: DE
  mode: prefix
  limit: 20
  select: url,top_keyword,top_keyword_volume,sum_traffic,keywords,referring_domains
  where: {"field":"url","is":["isubstring","ratgeber"]}  ← anpassen an gefundenen Subfolder
```

### Schritt 4: Keyword-Potenzial bewerten

Für die Top 5 Content-Seiten: Hole die organischen Keywords:

```
Tool: ahref:site-explorer-organic-keywords
Parameter:
  target: [URL der jeweiligen Seite, OHNE https://]
  date: [aktuelles Datum, Format YYYY-MM-DD] ← PFLICHT!
  country: DE
  mode: exact
  limit: 10
  select: keyword,volume,best_position,sum_traffic,keyword_difficulty
```

**Wichtig**:
- `date` ist ein Pflichtfeld bei diesem Endpoint!
- `target` OHNE https:// Prefix angeben
- Verwende `best_position` (nicht `position`) und `sum_traffic` (nicht `traffic`)

### Schritt 5: Ergebnisse zusammenstellen

Erstelle eine priorisierte Liste nach diesem Scoring:

```
Score = (sum_traffic × 2) + (keywords × 1.5) + (top_keyword_volume × 1) + (referring_domains × 10)
```

Priorisiere Themen die:
1. Hohes Suchvolumen haben
2. Viele Keywords abdecken (Content-Tiefe)
3. Wenige Referring Domains haben (leichter zu schlagen)
4. Thematisch zur Ziel-Website passen (prüfe gegen `seo.primary_topics` aus der Site-Config)

### Schritt 6: Output generieren

Erstelle `competitor-research-results.json`:

```json
{
  "analysis_date": "YYYY-MM-DD",
  "competitor_domain": "example.com",
  "domain_overview": {
    "organic_traffic": 12000,
    "organic_keywords": 1500,
    "traffic_value_usd": 5000
  },
  "content_subfolder": "/ratgeber/",
  "top_content_pages": [
    {
      "rank": 1,
      "url": "https://example.com/ratgeber/thema/",
      "top_keyword": "haupt-keyword",
      "top_keyword_volume": 400,
      "total_traffic": 1500,
      "total_keywords": 250,
      "referring_domains": 5,
      "score": 3825,
      "relevance_to_site": "hoch|mittel|niedrig",
      "topic_opportunity": "Kurze Beschreibung warum dieses Thema interessant ist"
    }
  ],
  "recommendations": [
    {
      "priority": 1,
      "topic": "Thema XYZ",
      "reason": "Hohes Volumen, wenig Backlinks nötig, passt zu unserem Sortiment",
      "suggested_keyword": "keyword vorschlag",
      "estimated_difficulty": "leicht|mittel|schwer"
    }
  ]
}
```

---

## Wichtige Hinweise

### API-Kosten beachten
- `site-explorer-metrics`: ~50 Units
- `site-explorer-top-pages`: ~28 Units pro Zeile mit referring_domains
- `site-explorer-organic-keywords`: ~variabel
- **Budget pro Analyse**: Ca. 1.000-2.000 Units für eine vollständige Konkurrenz-Analyse

### Fehlerbehandlung
- Wenn `site-explorer-top-pages` mit `order_by` fehlschlägt: Ohne `order_by` aufrufen
- Wenn eine Domain wenige Daten hat: Auf `mode: domain` statt `subdomains` wechseln
- Wenn der Content-Subfolder nicht identifizierbar ist: Alle Top-Seiten nach URL-Länge > 3 Segmente filtern (längere URLs = wahrscheinlich Content)

### Mehrere Konkurrenten
Wenn mehrere Domains analysiert werden sollen:
1. Für jede Domain Schritte 1-5 durchführen
2. Ergebnisse in einer Gesamtliste zusammenführen
3. Duplikate/ähnliche Themen gruppieren
4. Übergreifendes Ranking erstellen

---

## Nächster Schritt
Output wird an **Modul 2: SEO Outline** übergeben.
Der User oder Orchestrator wählt 1-3 Themen aus den Empfehlungen aus.
