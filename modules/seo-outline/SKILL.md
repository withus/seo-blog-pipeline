---
name: seo-outline
description: Erstellt ein SEO-optimiertes Artikel-Outline (H1 + H2s) basierend auf Konkurrenz-Analyse, SERP-Daten und Keyword-Recherche. Nutzt Ahrefs MCP, GSC MCP und Web Fetch.
---

# Modul 2: SEO Outline

## Wann diesen Skill verwenden
Wenn ein SEO-optimiertes Outline für einen neuen oder überarbeiteten Blogartikel erstellt werden soll. Kommt nach Modul 1 (Competitor Research) oder direkt mit einem Ziel-Keyword.

## Inputs

### Für neuen Artikel (vom Orchestrator "New Article")
- **Pflicht**: `competitor-research-results.json` (Output von Modul 1)
- **Pflicht**: Gewähltes Thema / Ziel-Keyword aus den Empfehlungen
- **Optional**: Zusätzliche Vorgaben vom User (Fokus, Zielgruppe, Produktbezug)

### Für Artikel-Überarbeitung (vom Orchestrator "Revise Article")
- **Pflicht**: Bestehender Artikel (URL oder Markdown)
- **Pflicht**: `gap-analysis.json` (vom Revise-Orchestrator generiert)
- **Optional**: GSC Performance-Daten

## Output
`outline.json` – Strukturiertes Outline mit:
- Optimiertes H1
- Alle H2s mit Keyword-Zuordnung
- Briefing pro Abschnitt (was soll rein, welche Fragen beantworten)
- Keyword-Map (Haupt-Keyword, Neben-Keywords, LSI-Terms)
- Suchintent-Analyse
- Empfohlene interne Verlinkungen

---

## Ablauf

### Schritt 1: Ziel-Keyword validieren und erweitern

Verwende `ahref:keywords-explorer-overview` für das Ziel-Keyword:

```
Tool: ahref:keywords-explorer-overview
Parameter:
  keywords: [ziel-keyword] ← Komma-separiert für mehrere
  country: de
  select: keyword,volume,difficulty,cpc,traffic_potential,parent_topic,intents
```

**Wichtig**: Feldnamen sind `difficulty` (nicht `keyword_difficulty`) und `intents` gibt ein Objekt mit `informational`, `commercial`, `transactional` etc. zurück.

Dann verwandte Keywords holen:

```
Tool: ahref:keywords-explorer-related-terms
Parameter:
  keywords: [ziel-keyword]
  country: de
  select: keyword,volume,difficulty,traffic_potential
  limit: 20
```

Und Suchvorschläge:

```
Tool: ahref:keywords-explorer-search-suggestions
Parameter:
  keywords: [ziel-keyword]
  country: de
  select: keyword,volume,difficulty
  limit: 15
```

**Ergebnis**: Eine Keyword-Map mit:
- 1 Haupt-Keyword (höchstes Volumen + erreichbare Difficulty)
- 5-10 Neben-Keywords (verwandte Terms mit eigenem Volumen)
- 10-15 LSI-Terms (Long-Tail, Fragen, Variationen)

### Schritt 2: SERP analysieren

Verwende `ahref:serp-overview` um zu sehen, wer aktuell rankt:

```
Tool: ahref:serp-overview
Parameter:
  keyword: [haupt-keyword]
  country: de
  select: url,title,position,traffic,keywords,backlinks
  top_positions: 10
```

**Analysiere**:
- Welche Content-Typen ranken? (Ratgeber, Listen, How-To, Vergleiche)
- Wie lang sind die rankenden Artikel? (→ unserer sollte mindestens genauso umfassend sein)
- Welche H1s verwenden die Top-Ergebnisse?
- Gibt es Featured Snippets oder People Also Ask?

### Schritt 3: Top-Konkurrenz-Artikel lesen

Verwende `web_fetch` um die Top 3 rankenden Artikel komplett zu lesen:

```
Tool: web_fetch
Parameter:
  url: [URL aus SERP-Analyse]
```

**Für jeden Artikel extrahieren**:
- H1 (Title)
- Alle H2s und H3s (Outline-Struktur)
- Kernthemen pro Abschnitt
- Unique Angles (was macht dieser Artikel anders?)
- Schwächen (was fehlt? was ist veraltet? was ist zu dünn?)

### Schritt 4: Eigene Performance prüfen (falls vorhanden)

Wenn die Ziel-Website bereits für verwandte Keywords rankt (GSC-Property aus Site-Config: `seo.gsc_property`):

```
Tool: gsc:search_analytics
Parameter:
  siteUrl: [seo.gsc_property aus Site-Config]
  startDate: [90 Tage zurück, YYYY-MM-DD]
  endDate: [heute, YYYY-MM-DD]
  dimensions: query,page
  queryFilter: [ziel-keyword oder Varianten]
  filterOperator: contains
  rowLimit: 20
```

**Nutze diese Daten um**:
- Bestehende Rankings nicht zu kannibalisieren
- Keywords zu identifizieren, für die wir schon Sichtbarkeit haben
- Interne Verlinkungen zu planen

### Schritt 5: Outline erstellen

Basierend auf allen gesammelten Daten:

#### H1 erstellen
- Enthält das Haupt-Keyword
- Ist besser/klarer als die H1s der Konkurrenz
- Spricht den Suchintent direkt an
- Max 60 Zeichen (für SERP-Anzeige)

#### H2s definieren
Für jede H2:
1. **Keyword-Zuordnung**: Welches Neben-Keyword deckt diese H2 ab?
2. **Suchintent**: Welche Frage beantwortet dieser Abschnitt?
3. **Briefing**: Was MUSS in diesen Abschnitt rein?
4. **Differenzierung**: Was machen wir BESSER als die Konkurrenz hier?
5. **Wortanzahl-Empfehlung**: Ca. 150-300 Wörter pro H2

#### Reihenfolge der H2s
Die H2s sollten einem logischen Lesefluss folgen:
1. Was ist es? (Definition/Einführung)
2. Warum ist es wichtig? (Problem/Nutzen)
3. Wie funktioniert es? (Erklärung/Anleitung)
4. Welche Varianten gibt es? (Vergleich/Typen)
5. Worauf muss ich achten? (Tipps/Fehler vermeiden)
6. Wie mache ich es selbst? (DIY/Anleitung)
7. Was kostet es? (Preis/Wirtschaftlichkeit) – falls relevant
8. FAQ (häufige Fragen – gut für Featured Snippets)

Nicht jeder Artikel braucht alle Punkte – die Auswahl hängt vom Suchintent ab.

### Schritt 6: Output generieren

Erstelle `outline.json`:

```json
{
  "meta": {
    "created_date": "YYYY-MM-DD",
    "target_domain": "[domain aus Site-Config]",
    "competitor_source": "competitor-research-results.json",
    "mode": "new_article | revision"
  },
  "keyword_map": {
    "primary_keyword": {
      "keyword": "sonnensegel befestigen",
      "volume": 500,
      "difficulty": 15,
      "intent": "informational"
    },
    "secondary_keywords": [
      {"keyword": "sonnensegel aufhängen", "volume": 200, "difficulty": 10},
      {"keyword": "sonnensegel spannen", "volume": 150, "difficulty": 8}
    ],
    "lsi_terms": [
      "sonnensegel hauswand",
      "sonnensegel montage",
      "sonnensegel pfosten",
      "sonnensegel ohne bohren"
    ],
    "questions_to_answer": [
      "Wie befestige ich ein Sonnensegel an der Hauswand?",
      "Welche Befestigung für ein Sonnensegel?",
      "Kann man ein Sonnensegel ohne Bohren aufhängen?",
      "Wie groß muss ein Sonnensegel für eine Terrasse sein?"
    ]
  },
  "serp_analysis": {
    "dominant_content_type": "how-to / anleitung",
    "avg_word_count_top3": 1500,
    "featured_snippet_opportunity": true,
    "top3_h1s": [
      "Sonnensegel befestigen: So geht's richtig",
      "...",
      "..."
    ],
    "content_gaps_found": [
      "Keiner der Top-3 erklärt verschiedene Segelformen und deren Eignung",
      "Kein Artikel zeigt Vorher/Nachher-Vergleiche",
      "Mieterwohnung wird kaum thematisiert (ohne Bohren)"
    ]
  },
  "outline": {
    "h1": "Sonnensegel befestigen: Alle Methoden einfach erklärt",
    "meta_description": "Du willst ein Sonnensegel befestigen? Hier findest du alle Methoden – mit und ohne Bohren, für Mieter und Eigentümer. Einfache Anleitung.",
    "estimated_word_count": 1800,
    "sections": [
      {
        "h2": "Warum ein Sonnensegel auf der Terrasse wirklich etwas bringt",
        "target_keyword": "sonnensegel terrasse",
        "intent": "Suchintent abholen: Nutzer will wissen ob es sich lohnt",
        "briefing": "Kurz erklären wie ein Sonnensegel Schatten spendet. Keine Physik-Vorlesung, sondern praktischer Nutzen: kühlerer Sitzplatz, Schutz vor UV-Licht, mehr Aufenthaltsqualität. Typische Anwendungsfälle: Terrasse, Balkon, Spielbereich, Sitzecke im Garten.",
        "word_count": 200,
        "differentiation": "Ehrliche Erwartungshaltung setzen: was ein Sonnensegel NICHT kann (Schutz bei Starkregen)"
      },
      {
        "h2": "Welches Sonnensegel eignet sich wofür?",
        "target_keyword": "sonnensegel formen",
        "intent": "Commercial – Nutzer will vergleichen vor dem Kauf",
        "briefing": "Dreieck vs. Rechteck vs. Quadrat, wasserdurchlässig vs. wasserabweisend. Für jeden Typ: wofür geeignet, Stärken/Schwächen. Kurze Tabelle möglich.",
        "word_count": 250,
        "differentiation": "CONTENT GAP: Keiner der Top-3 Konkurrenten erklärt die Unterschiede der Segelformen"
      },
      {
        "h2": "Sonnensegel an der Hauswand befestigen – die beliebteste Methode",
        "target_keyword": "sonnensegel hauswand",
        "intent": "Transactional/How-To – Nutzer will es jetzt machen",
        "briefing": "Schritt-für-Schritt: Untergrund prüfen, passende Befestigung wählen (Wandhaken, Augschrauben, Fassadenhalter), Segel einhängen, spannen. Fehler vermeiden.",
        "word_count": 300,
        "differentiation": "Detaillierter als Konkurrenz: konkrete Befestigungsempfehlungen, Spannwinkel, Hinweise zur Wandart"
      },
      {
        "h2": "Sonnensegel aufhängen ohne Bohren – perfekt für Mietwohnungen",
        "target_keyword": "sonnensegel ohne bohren",
        "intent": "Informational – große Zielgruppe der Mieter",
        "briefing": "Klemmstangen, Teleskopmasten, Standfüße mit Gewichten. Rückstandsfrei entfernbar. Besonders für Mieter relevant.",
        "word_count": 250,
        "differentiation": "CONTENT GAP: Mieter-Perspektive wird von Konkurrenz kaum behandelt"
      },
      {
        "h2": "Sonnensegel an Pfosten befestigen",
        "target_keyword": "sonnensegel pfosten",
        "intent": "Spezifische Anleitung für schwierigere Montage",
        "briefing": "Besonderheiten bei Pfostenmontage: Zugkräfte, Sicherheit, geeignete Fundamente. Einbetonierte Pfosten vs. Bodenhülsen.",
        "word_count": 200,
        "differentiation": "Sicherheitshinweise die Konkurrenz auslässt"
      },
      {
        "h2": "So findest du die richtige Größe und Aufhängehöhe",
        "target_keyword": "sonnensegel größe berechnen",
        "intent": "Praktische Hilfe vor dem Kauf",
        "briefing": "Faustregel: Das Segel kleiner wählen als den Abstand zwischen den Befestigungspunkten, weil Beschläge und Spannseile Platz brauchen. Erklärung warum die Neigung wichtig ist (Wasserablauf). Sonnenstand beachten.",
        "word_count": 200,
        "differentiation": "Konkrete Berechnungshilfe die Konkurrenten nicht bieten"
      },
      {
        "h2": "Häufige Fehler beim Sonnensegel befestigen",
        "target_keyword": "sonnensegel fehler",
        "intent": "Informational – Nutzer will Fehler vermeiden",
        "briefing": "Top 5 Fehler: Zu schwache Befestigung, zu straff oder zu lose gespannt, falsche Neigung, Wind ignorieren, falsche Erwartungen. Pro Fehler kurze Lösung.",
        "word_count": 200,
        "differentiation": "Ehrlicher Inhalt der Vertrauen aufbaut"
      },
      {
        "h2": "FAQ: Die häufigsten Fragen zu Sonnensegeln",
        "target_keyword": "sonnensegel fragen",
        "intent": "Featured Snippet Opportunity",
        "briefing": "5-7 kurze Fragen und Antworten. Aus 'People Also Ask' und Keyword-Recherche. Jede Antwort max 2-3 Sätze. Strukturiert für FAQ-Schema-Markup.",
        "word_count": 200,
        "differentiation": "FAQ-Schema für Featured Snippets"
      }
    ]
  },
  "internal_links": {
    "note": "Keine konkreten URLs – Modul 6 (Internal Linking) löst diese zu echten, validierten URLs auf",
    "suggested_topics": [
      {
        "anchor_text_suggestion": "Sonnensegel in verschiedenen Größen",
        "link_target_topic": "Sonnensegel Kategorie oder Produktseite",
        "placement": "Im Abschnitt 'Welches Sonnensegel eignet sich wofür'"
      },
      {
        "anchor_text_suggestion": "Befestigungsmaterial für die Hauswand",
        "link_target_topic": "Zubehör Kategorie oder Produktseite",
        "placement": "Im Abschnitt 'Sonnensegel an der Hauswand befestigen'"
      },
      {
        "anchor_text_suggestion": "Sonnenschirme als Alternative",
        "link_target_topic": "Sonnenschirm Kategorie",
        "placement": "Im Abschnitt 'Richtige Größe und Aufhängehöhe'"
      }
    ]
  },
  "image_suggestions": [
    {
      "placement": "Nach H2 'Welches Sonnensegel eignet sich wofür'",
      "description": "Vergleichsbild: 4 verschiedene Segelformen nebeneinander",
      "alt_text": "Vergleich von Dreieck-, Rechteck-, Quadrat- und Trapezsegel"
    },
    {
      "placement": "Im Abschnitt 'Sonnensegel an der Hauswand befestigen'",
      "description": "Step-by-Step Bilder: Haken montieren, Segel einhängen",
      "alt_text": "Sonnensegel mit Wandhaken an der Hauswand befestigen – Schritt für Schritt"
    },
    {
      "placement": "Im Abschnitt 'Richtige Größe und Aufhängehöhe'",
      "description": "Draufsicht Terrassenskizze: Anordnung der Befestigungspunkte",
      "alt_text": "Passende Anordnung der Befestigungspunkte für ein Sonnensegel auf der Terrasse"
    }
  ]
}
```

---

## Spezial-Modus: Revision-Outline

Wenn ein bestehender Artikel überarbeitet wird, ändert sich der Ablauf:

### Zusätzlicher Input
- `gap-analysis.json` vom Revise-Orchestrator enthält:
  - Keywords für die der Artikel schlecht rankt (Position 11-50)
  - Keywords die die Konkurrenz hat, wir aber nicht
  - Abschnitte die zu dünn sind

### Angepasster Schritt 5
Statt komplett neues Outline:
1. Bestehende H2-Struktur als Basis nehmen
2. Markieren welche H2s bleiben, welche angepasst werden, welche neu dazukommen
3. Für jede H2 angeben: `"action": "keep" | "revise" | "new" | "merge" | "remove"`

```json
{
  "sections": [
    {
      "h2": "Bestehende Überschrift",
      "action": "revise",
      "revision_notes": "Keyword X einbauen, Abschnitt um 100 Wörter erweitern, veraltete Info zu Y aktualisieren",
      "target_keyword": "neues-keyword",
      "word_count": 300
    },
    {
      "h2": "Komplett neue Überschrift",
      "action": "new",
      "briefing": "Dieses Thema fehlt komplett im bestehenden Artikel. Konkurrenz deckt es ab.",
      "target_keyword": "neues-keyword",
      "word_count": 250
    }
  ]
}
```

---

## Qualitätskriterien für das Outline

Bevor das Outline an Modul 3 übergeben wird, prüfe:

- [ ] H1 enthält das Haupt-Keyword und ist ≤ 60 Zeichen
- [ ] Meta-Description ist ≤ 155 Zeichen und enthält Haupt-Keyword
- [ ] Jede H2 hat ein zugeordnetes Keyword
- [ ] Jede H2 hat ein klares Briefing
- [ ] Mindestens 1 Content Gap der Konkurrenz wird adressiert
- [ ] Geschätzte Gesamtwortzahl ist ≥ Durchschnitt der Top 3 SERP-Ergebnisse
- [ ] Interne Verlinkungen als Themen-Vorschläge definiert (keine echten URLs – das macht Modul 6)
- [ ] Bild-Platzierungen sind definiert

---

## Nächster Schritt
Output wird an **Modul 3: Article Writer** übergeben.
