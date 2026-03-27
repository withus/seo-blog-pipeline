# SEO Blog Skill System – Architektur

## Übersicht

Modulares System zur automatisierten Erstellung und Überarbeitung von SEO-optimierten Blogartikeln für beliebige Online-Shops/Websites. Domain-agnostisch – neue Websites werden über Site-Configs eingebunden.

## Architektur

```
                    User-Anfrage
                         │
                         ▼
                   ┌──────────┐
                   │  ROUTER   │  ← Einstiegspunkt
                   └─────┬────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
    ┌──────────────┐ ┌──────────┐ ┌──────────┐
    │ Orchestrator │ │Orchestr. │ │  Modul   │
    │ New Article  │ │ Revise   │ │  direkt  │
    └──────┬───────┘ └────┬─────┘ └──────────┘
           │              │
           ▼              ▼
    ┌─────────────────────────────────────┐
    │         Module 1-6 (verkettet)       │
    │                                      │
    │  M1 Competitor Research              │
    │    ↓                                 │
    │  M2 SEO Outline                      │
    │    ↓                                 │
    │  M3 Article Writer  ←──┐             │
    │    ↓                    │             │
    │  M4 Quality Check  ────┘ (max 2x)   │
    │    ↓                                 │
    │  M5 Image Generation                 │
    │    ↓                                 │
    │  M6 Internal Linking                 │
    └─────────────────────────────────────┘
           │
           ▼
    Fertiges Artikel-Paket
```

## Dateistruktur

```
seo-blog-pipeline/
├── ROUTER.md                           ← Einstiegspunkt (immer zuerst lesen)
├── ARCHITECTURE.md                     ← Diese Datei
│
├── config/
│   ├── SITE-CONFIG.md                  ← Template + Anleitung für neue Sites
│   ├── style-guide.md                  ← Globale Stil-Regeln
│   ├── blacklist-words.md              ← Verbotene Wörter/Phrasen
│   └── sites/
│       └── example-shop-example.json   ← Beispiel Site-Config
│
├── orchestrators/
│   ├── new-article/SKILL.md            ← Neuen Artikel erstellen
│   └── revise-article/SKILL.md         ← Bestehenden Artikel überarbeiten
│
└── modules/
    ├── competitor-research/SKILL.md    ← Modul 1: Konkurrenz analysieren
    ├── seo-outline/SKILL.md            ← Modul 2: Outline erstellen
    ├── article-writer/SKILL.md         ← Modul 3: Artikel schreiben
    ├── quality-check/SKILL.md          ← Modul 4: Qualitätsprüfung
    ├── image-generation/SKILL.md       ← Modul 5: Bilder generieren
    └── internal-linking/SKILL.md       ← Modul 6: Links validieren
```

## Input/Output-Kette

| Schritt | Modul | Input | Output |
|---------|-------|-------|--------|
| 0 | Router | User-Anfrage | → Orchestrator-Wahl |
| 0.5 | Site-Config | Domain | → `sites/[domain].json` |
| 1 | Competitor Research | Site-Config + Competitor-Domain(s) | → `competitor-research-results.json` |
| 2 | SEO Outline | Results + Keyword + Site-Config | → `outline.json` |
| 3 | Article Writer | Outline + Style Guide + Blacklist | → `article.md` + `article-meta.json` |
| 4 | Quality Check | Article + Outline + Configs | → `quality-report.json` + ggf. `revision-notes.md` |
| 4→3 | Feedback-Loop | revision-notes.md | → korrigierte `article.md` (max 2x) |
| 5 | Image Generation | APPROVED article.md + Outline | → `article-with-images.md` + `image-manifest.json` + PNGs |
| 6 | Internal Linking | article-with-images.md + Site-Config | → `article-linked.md` (FINAL) + `link-report.json` |

## Schlüsselprinzipien

**Domain-agnostisch**: Keine hartcodierten Domains. Alles läuft über Site-Configs.

**Keine erfundenen URLs**: Modul 3 setzt `[INTERNER_LINK: Ankertext | Thema]` Platzhalter. Erst Modul 6 crawlt die echte Website und setzt validierte URLs ein.

**Quality-Gate**: Kein Artikel geht an Modul 5/6 ohne Quality-Check Freigabe (Score ≥90 oder max 2 Loops).

**User-Stopps**: Der Orchestrator hält an 2 Stellen für User-Entscheidungen:
1. Nach Modul 1: Keyword-/Themenwahl
2. Nach Modul 2: Outline-Freigabe

## Externe Tools & APIs

| Tool | Verwendet in |
|------|-------------|
| Ahrefs MCP | Modul 1 (site-explorer), Modul 2 (keywords-explorer, serp-overview) |
| GSC MCP | Modul 2 (search_analytics), Revise-Orchestrator (performance) |
| web_fetch | Modul 2 (Konkurrenz lesen), Modul 6 (Sitemap, Validation) |
| web_search | Modul 3 (Fakten-Recherche), Modul 4 (Faktencheck) |
| nano-banana-pro | Modul 5 (generate_image, describe_image) |

## Status
- Router
- Orchestrator: New Article
- Orchestrator: Revise Article
- Modul 1: Competitor Research
- Modul 2: SEO Outline
- Modul 3: Article Writer
- Modul 4: Quality Check (mit Feedback-Loop)
- Modul 5: Image Generation
- Modul 6: Internal Linking
- Config: Site-Config Template + Beispiel
- Config: Style Guide
- Config: Blacklist
