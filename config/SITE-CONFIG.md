# Site Configuration

Dieses File wird pro Website/Shop erstellt und von allen Modulen gelesen. Es ersetzt hartcodierte Domain-Referenzen und macht das gesamte System domain-agnostisch.

**Speicherort**: `/config/sites/[domain-slug].json`
**Beispiel**: `/config/sites/example-shop-example.json`

---

## Template

```json
{
  "site": {
    "domain": "example-shop.example",
    "name": "example-shop",
    "language": "de",
    "country_code": "de",
    "protocol": "https"
  },

  "content": {
    "blog_path_patterns": ["/ratgeber/", "/blog/", "/magazin/"],
    "category_path_patterns": ["/katalog/", "/kategorie/", "/c/"],
    "product_path_patterns": ["/produkt/", "/p/", "/shop/"],
    "sitemap_url": null,
    "robots_txt_url": null
  },

  "seo": {
    "primary_topics": [
      "Gartenmöbel",
      "Sonnenschirme",
      "Terrassen",
      "Loungemöbel",
      "Pflege"
    ],
    "brand_terms": ["example-shop", "example-shop.example"],
    "gsc_property": "sc-domain:example-shop.example",
    "ahrefs_target": "example-shop.example"
  },

  "style": {
    "tone": "du-form",
    "style_guide": "/config/style-guide.md",
    "blacklist": "/config/blacklist-words.md"
  },

  "competitors": [
    {
      "domain": "garten-beispiel-eins.example",
      "note": "Direkter Wettbewerber, ähnliches Sortiment"
    },
    {
      "domain": "garten-beispiel-zwei.example",
      "note": "Großer Gartenmöbel-Shop"
    }
  ],

  "internal_linking": {
    "max_links_per_article": 7,
    "min_links_per_article": 3,
    "prefer_categories_over_products": true,
    "known_urls": []
  }
}
```

---

## Wie die Module die Site-Config nutzen

### Modul 1: Competitor Research
- Liest `seo.primary_topics` für die Relevanz-Bewertung der Konkurrenz-Artikel
- Liest `competitors[]` als Standard-Domains wenn kein Competitor angegeben wird
- Liest `site.country_code` für Ahrefs-Ländercode

### Modul 2: SEO Outline
- Liest `seo.gsc_property` für GSC-Abfragen
- Liest `seo.primary_topics` um Content Gaps auf Relevanz zu prüfen
- Liest `content.blog_path_patterns` um bestehende Artikel zu identifizieren
- Setzt **keine konkreten internen Links** mehr – das macht Modul 6

### Modul 3: Article Writer
- Liest `style.tone` (du-form / sie-form / neutral)
- Liest `style.style_guide` und `style.blacklist` Pfade
- Setzt **Platzhalter-Links** statt echte URLs: `[INTERNER_LINK: Gartentisch Teakholz]`
- Der Ankertext wird vorgeschlagen, die URL bleibt offen für Modul 6

### Modul 4: Quality Check
- Prüft ob Platzhalter-Links vorhanden sind (statt echte URLs zu erwarten)
- Liest `style.*` für Stil-Prüfungen
- SEO-Check nutzt `site.country_code`

### Modul 5: Image Generation
- Keine direkten Site-Config Abhängigkeiten
- Kann `site.name` für Branding nutzen falls gewünscht

### Modul 6: Internal Linking
- Liest `site.domain` und `site.protocol` für URL-Aufbau
- Liest `content.sitemap_url` oder baut Standard-URL: `https://[domain]/sitemap.xml`
- Liest `content.*_path_patterns` für URL-Kategorisierung
- Liest `internal_linking.*` für Link-Regeln
- Liest `internal_linking.known_urls` als Cache (wird nach erstem Crawl befüllt)
- Ersetzt die `[INTERNER_LINK: ...]` Platzhalter durch echte, validierte URLs

---

## Neues Site-Config erstellen

Wenn ein neuer Shop/Website hinzukommt:

1. Kopiere das Template
2. Fülle die Felder aus:
   - Domain und Sprache
   - URL-Muster für Blog, Kategorien, Produkte (am besten vorher kurz die Website anschauen)
   - Primäre Themen (worum geht es im Shop?)
   - Bekannte Konkurrenten
3. Speichere unter `/config/sites/[domain-slug].json`
4. Beim ersten Durchlauf: Modul 6 crawlt die Sitemap und befüllt `known_urls`

---

## Beispiel: Zweiter Shop

```json
{
  "site": {
    "domain": "example.com",
    "name": "Beispiel Gartenwerkzeug",
    "language": "de",
    "country_code": "de",
    "protocol": "https"
  },
  "content": {
    "blog_path_patterns": ["/blog/"],
    "category_path_patterns": ["/collections/", "/kategorie/"],
    "product_path_patterns": ["/products/"],
    "sitemap_url": "https://example.com/sitemap.xml",
    "robots_txt_url": null
  },
  "seo": {
    "primary_topics": [
      "Gartenwerkzeug",
      "Spaten",
      "Gartenschere",
      "Rasenpflege",
      "Gartenarbeit"
    ],
    "brand_terms": ["beispiel-gartenwerkzeug"],
    "gsc_property": "sc-domain:example.com",
    "ahrefs_target": "example.com"
  },
  "style": {
    "tone": "du-form",
    "style_guide": "/config/style-guide.md",
    "blacklist": "/config/blacklist-words.md"
  },
  "competitors": [
    {"domain": "werkzeug-beispiel.example", "note": "Fachhandel für Gartenwerkzeug"},
    {"domain": "garten-beispiel-drei.example", "note": "Breites Gartensortiment"}
  ],
  "internal_linking": {
    "max_links_per_article": 5,
    "min_links_per_article": 2,
    "prefer_categories_over_products": true,
    "known_urls": []
  }
}
```
