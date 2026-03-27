---
name: internal-linking
description: Crawlt die Ziel-Website (Sitemap, Kategorien, Blog-Artikel, Produktseiten) und ersetzt die Platzhalter-Links im Artikel durch echte, validierte interne Links. Kein Link wird gesetzt ohne vorherige URL-Validierung.
---

# Modul 6: Internal Linking

## Wann diesen Skill verwenden
Nach dem Quality Check (Modul 4) und Image Generation (Modul 5). Letzter Schritt bevor der Artikel veröffentlicht wird. Ersetzt alle internen Links durch echte, geprüfte URLs.

## Inputs
- **Pflicht**: `article-with-images.md` (APPROVED Artikel aus Modul 5, mit `[INTERNER_LINK: Ankertext | Thema]` Platzhaltern)
- **Pflicht**: `outline.json` (enthält internal_links.suggested_topics)
- **Pflicht**: Site-Config (`/config/sites/[domain].json`) – Domain, URL-Muster, Link-Regeln
- **Optional**: Bereits bekannte URL-Liste (aus Site-Config `internal_linking.known_urls`)

## Output
- `article-linked.md` – Artikel mit validierten internen Links
- `link-report.json` – Alle gefundenen, vorgeschlagenen und gesetzten Links

---

## Ablauf

### Schritt 1: Website-Struktur ermitteln

#### 1a: Robots.txt lesen

```
Tool: web_fetch
URL: https://[domain]/robots.txt
```

Aus der robots.txt extrahieren:
- Sitemap-URL(s)
- Disallow-Regeln (diese Pfade NICHT verlinken!)

#### 1b: Sitemap parsen

```
Tool: web_fetch
URL: [Sitemap-URL aus robots.txt]
```

Die Sitemap enthält alle indexierten URLs der Website. Parse das XML und erstelle eine Liste aller URLs.

**Bei Sitemap-Index** (mehrere Sitemaps):
- Erst den Index laden
- Dann die relevanten Sub-Sitemaps laden:
  - Kategorie-Sitemap
  - Produkt-Sitemap
  - Blog/Ratgeber-Sitemap
- Produkt-Sitemaps können sehr groß sein → nur die ersten 500 URLs laden

#### 1c: URL-Kategorisierung

Kategorisiere jede URL basierend auf den Pfadmustern aus der Site-Config (`content.*_path_patterns`):

| Typ | Erkennungsmuster (aus Site-Config) | Beispiel |
|-----|-----------------------------------|---------|
| Startseite | Nur Domain, kein Pfad | example.com/ |
| Kategorie | `content.category_path_patterns` | /katalog/gartentische/ |
| Produkt | `content.product_path_patterns` | /produkt/teak-gartentisch-oval |
| Blog/Ratgeber | `content.blog_path_patterns` | /ratgeber/gartenmoebel-reinigen |
| Info-Seiten | /ueber-uns, /kontakt, /agb, /impressum | /impressum |

**Relevanz-Filter**: Nur Kategorien, Produkte und Blog-Artikel für die Verlinkung berücksichtigen. Info-Seiten und Startseite ignorieren (außer ausdrücklich gewünscht).

### Schritt 2: Thematisches Matching

Für jeden Link-Kandidaten aus der URL-Liste:

#### 2a: URL-basiertes Matching
Vergleiche die URL-Slugs mit den Keywords aus `outline.json`:

```
Artikel-Keywords: gartenmöbel reinigen, teakholz, natron, terrasse, dampfreiniger
URL: /teakholz-gartentische/ → Match auf "teakholz" ✓
URL: /sonnenschirme/ → Kein direkter Keyword-Match, aber thematisch verwandt ✓
URL: /fahrradstaender/ → Kein Match ✗
```

#### 2b: Kontext-Matching (bei nicht eindeutigen URLs)
Wenn die URL allein nicht reicht, lade die Seite und prüfe den Titel und die Meta-Description:

```
Tool: web_fetch
URL: [Kandidaten-URL]
```

Extrahiere:
- `<title>` Tag
- `<meta name="description">` Tag
- H1-Überschrift
- Erste 200 Wörter des Inhalts

Vergleiche diese Informationen mit dem Thema des Artikels.

#### 2c: Relevanz-Score berechnen

Für jeden URL-Kandidaten:
- URL-Slug enthält ein Keyword aus dem Artikel: +3 Punkte
- Seitentitel enthält ein Keyword: +2 Punkte
- Meta-Description enthält ein Keyword: +1 Punkt
- Gleiche Produktkategorie wie Artikelthema: +2 Punkte
- Anderer Blog-Artikel zum verwandten Thema: +3 Punkte

Mindest-Score für eine Verlinkung: 3 Punkte.

### Schritt 3: Link-Validierung

**KRITISCH**: Jeder Link MUSS vor dem Einsetzen validiert werden.

```
Tool: web_fetch
URL: https://[domain][pfad]
```

Prüfe:
- [ ] Seite gibt HTTP 200 zurück (kein 404, kein Redirect-Loop)
- [ ] Seite ist NICHT in robots.txt Disallow
- [ ] Seite hat echten Inhalt (kein leerer Body, keine Wartungsseite)
- [ ] Seite ist thematisch wirklich relevant (nicht nur zufälliger Keyword-Match)

**Wenn die URL nicht erreichbar ist oder 404 zurückgibt**: Link NICHT setzen. In den Report als "failed_validation" aufnehmen.

### Schritt 4: Ankertext und Platzierung bestimmen

Für jeden validierten Link:

#### Ankertext-Regeln
- **Natürlich**: Muss sich organisch in den Satz einfügen
- **Beschreibend**: Nicht "hier klicken" oder "mehr erfahren"
- **Keyword-haltig**: Wenn möglich ein relevantes Keyword im Ankertext
- **Variiert**: Nicht immer denselben Ankertext für dieselbe URL
- **Kurz**: 2-5 Wörter ideal

Gute Ankertexte:
- "Gartentische aus Teakholz ansehen"
- "passenden Sonnenschirm bestellen"
- "verschiedene Tischgrößen vergleichen"

Schlechte Ankertexte:
- "hier klicken"
- "mehr dazu"
- "auf dieser Seite"
- "Gartentische aus Teakholz ansehen bei example-shop.example dem Experten für Gartenmöbel" (zu lang)

#### Platzierungs-Regeln
- Max/Min Links aus Site-Config `internal_linking.max_links_per_article` / `min_links_per_article`
- Mindestens **1 Link in den ersten 300 Wörtern**
- Nicht mehr als **2 Links pro H2-Abschnitt**
- Links zu **Kategorien/Produkten**: Im Fließtext, wo das Produkt erwähnt wird
- Links zu **anderen Blog-Artikeln**: Am Ende eines thematisch verwandten Abschnitts oder als "Lies auch"-Empfehlung
- **Niemals** im FAQ-Bereich Links setzen (stört das FAQ-Schema-Markup)

#### Link-Typen im Artikel

**Inline-Link** (im Fließtext):
```markdown
Wenn der Sonnenschirm nicht mehr sauber schließt, hilft nur noch ein
[neuer Sonnenschirm](/sonnenschirme/).
```

**Thematischer Verweis** (Ende eines Abschnitts):
```markdown
Mehr dazu in unserem Ratgeber: [Sonnensegel richtig befestigen](/ratgeber/sonnensegel-befestigen/)
```

### Schritt 5: Link-Platzhalter auflösen

Der Artikel enthält Platzhalter aus Modul 3:
```markdown
[INTERNER_LINK: Gartentische aus Teakholz | Gartentisch Kategorie]
```

Für jeden Platzhalter:
1. Extrahiere den Ankertext ("Gartentische aus Teakholz") und das Thema ("Gartentisch Kategorie")
2. Finde die beste passende URL aus den validierten Kandidaten (Schritt 2+3)
3. Wenn eine passende URL gefunden → Platzhalter ersetzen durch: `[Gartentische aus Teakholz](/katalog/gartentische)`
4. Wenn keine passende URL gefunden → Platzhalter entfernen und den Satz ohne Link belassen
5. Ankertext beibehalten wenn er gut ist, sonst leicht anpassen

### Schritt 6: Zusätzliche Links ergänzen

Basierend auf dem thematischen Matching (Schritt 2):
- Gibt es relevante Kategorien die noch nicht verlinkt sind?
- Gibt es bestehende Blog-Artikel zum verwandten Thema?
- Gibt es Produktseiten die im Kontext Sinn machen?

Füge diese Links an passenden Stellen ein, unter Beachtung der Platzierungs-Regeln.

### Schritt 7: Output generieren

**article-linked.md**: Artikel mit allen validierten Links

**link-report.json**:
```json
{
  "meta": {
    "checked_date": "2026-02-19",
    "domain": "example-shop.example",
    "article": "gartenmoebel-reinigen"
  },
  "sitemap_stats": {
    "total_urls_found": 300,
    "categories": 25,
    "products": 220,
    "blog_articles": 15,
    "other": 40
  },
  "links_set": [
    {
      "url": "/gartentische/",
      "anchor_text": "Gartentische aus Teakholz ansehen",
      "placement": "Abschnitt 'Wann austauschen', Absatz 2",
      "type": "category",
      "validation": "passed",
      "relevance_score": 5
    },
    {
      "url": "/sonnenschirme/",
      "anchor_text": "neuen Sonnenschirm",
      "placement": "Abschnitt 'Wann austauschen', Absatz 3",
      "type": "category",
      "validation": "passed",
      "relevance_score": 5
    },
    {
      "url": "/gartenstuehle/",
      "anchor_text": "Gartenstühle in verschiedenen Farben",
      "placement": "Abschnitt 'Materialien richtig pflegen', nach Materialübersicht",
      "type": "category",
      "validation": "passed",
      "relevance_score": 4
    },
    {
      "url": "/ratgeber/sonnensegel-befestigen/",
      "anchor_text": "Sonnensegel richtig befestigen",
      "placement": "Abschnitt 'Materialien richtig pflegen', Verweis am Ende",
      "type": "blog_article",
      "validation": "passed",
      "relevance_score": 3
    }
  ],
  "links_rejected": [
    {
      "url": "/sitzauflagen/",
      "reason": "URL gibt 404 zurück",
      "validation": "failed_404"
    }
  ],
  "links_from_outline_validated": {
    "/gartentische/": "passed",
    "/sonnenschirme/": "passed"
  },
  "disallow_paths": [
    "/admin/",
    "/checkout/",
    "/warenkorb/"
  ]
}
```

---

## Spezialfälle

### Große Shops (>1.000 Produkte)
Bei sehr großen Shops nicht alle Produkt-URLs laden. Stattdessen:
1. Nur Kategorie-URLs laden (sind weniger)
2. Nur Blog/Ratgeber-URLs laden
3. Gezielt nach Produkten suchen die zum Artikelthema passen

### Kein Sitemap vorhanden
Wenn robots.txt keine Sitemap enthält:
1. Probiere Standard-Pfade: `/sitemap.xml`, `/sitemap_index.xml`
2. Wenn nichts gefunden: Startseite laden und Navigation extrahieren
3. Hauptkategorien aus der Navigation als Link-Kandidaten verwenden

### Mehrsprachige Shops
Achte auf die Sprachversion (aus Site-Config `site.language`):
- Nur URLs in der Zielsprache verwenden
- Prüfe hreflang-Tags wenn vorhanden

---

## API-Kosten
- 1 web_fetch für robots.txt
- 1-5 web_fetch für Sitemap(s)
- 3-10 web_fetch für Link-Validierung (pro Kandidat)
- Typisch: 8-20 web_fetch Aufrufe pro Artikel
- Keine Ahrefs-Kosten

---

## Nächster Schritt
Output ist der finale Artikel. Geht an den Orchestrator zur Zusammenstellung des Gesamtpakets.
