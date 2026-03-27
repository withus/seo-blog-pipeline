---
name: seo-blog-router
description: Einstiegspunkt für das SEO-Blog-System. Analysiert den User-Request und leitet an den richtigen Orchestrator weiter. Prüft ob eine Site-Config existiert und hilft bei der Erstellung wenn nötig.
---

# SEO Blog Router

## Wann diesen Skill verwenden
Dies ist der **Haupteinstiegspunkt** für alle SEO-Blog-Aufgaben. Lies dieses File ZUERST wenn der User nach Blog-Content, SEO-Artikeln oder Content-Optimierung fragt.

---

## Schritt 1: Request klassifizieren

Analysiere die User-Anfrage und ordne sie einem dieser Pfade zu:

### Pfad A → Orchestrator "New Article"
**Trigger-Phrasen**:
- "Schreib einen neuen Artikel über..."
- "Erstelle einen Blogartikel zu..."
- "Neuer Ratgeber für..."
- "Analysiere die Konkurrenz und erstelle Content für..."
- "Welche Blog-Themen sollten wir für [Domain] angehen?"
- "Content für [Keyword] erstellen"

**Erkennungsmerkmal**: Kein bestehender Artikel vorhanden. User will etwas Neues.

→ Lies `/orchestrators/new-article/SKILL.md`

---

### Pfad B → Orchestrator "Revise Article"
**Trigger-Phrasen**:
- "Überarbeite diesen Artikel: [URL]"
- "Dieser Artikel rankt schlecht..."
- "Aktualisiere den Ratgeber zu..."
- "Mach eine Content-Gap-Analyse für [URL]"
- "Was können wir an [URL] verbessern?"
- "Der Artikel ist veraltet, bitte updaten"

**Erkennungsmerkmal**: Bestehender Artikel (URL oder Text) wird genannt. User will etwas Bestehendes verbessern.

→ Lies `/orchestrators/revise-article/SKILL.md`

---

### Pfad C → Einzelnes Modul direkt
Manchmal will der User nur einen einzelnen Schritt, nicht den kompletten Ablauf:

| User-Anfrage | Modul |
|-------------|-------|
| "Analysiere die Konkurrenz von [domain]" | Modul 1: `/modules/competitor-research/SKILL.md` |
| "Erstelle ein Outline für [Keyword]" | Modul 2: `/modules/seo-outline/SKILL.md` |
| "Schreib den Artikel basierend auf diesem Outline" | Modul 3: `/modules/article-writer/SKILL.md` |
| "Prüfe diesen Artikel auf Qualität" | Modul 4: `/modules/quality-check/SKILL.md` |
| "Generiere Bilder für diesen Artikel" | Modul 5: `/modules/image-generation/SKILL.md` |
| "Prüfe die internen Links in diesem Artikel" | Modul 6: `/modules/internal-linking/SKILL.md` |

---

## Schritt 2: Site-Config prüfen

Bevor du an einen Orchestrator oder ein Modul übergibst, prüfe ob eine Site-Config existiert.

**Prüfe**:
```
Pfad: /config/sites/[domain-slug].json
```

Dabei wird der Domain-Slug aus der Domain erzeugt: Punkte → Bindestriche.
Beispiel: `example-shop.example` → `example-shop-example.json`

### Wenn Site-Config existiert:
→ Weiter zum gewählten Orchestrator/Modul. Die Config wird automatisch geladen.

### Wenn KEINE Site-Config existiert:
Frage den User nach den nötigsten Informationen:

> "Für [Domain] habe ich noch keine Konfiguration. Ich brauche ein paar Infos:
>
> 1. **Sprache**: Deutsch? Andere?
> 2. **Anrede**: Du-Form oder Sie-Form?
> 3. **Hauptthemen**: Was sind die 3-5 wichtigsten Themen/Produktbereiche?
> 4. **Konkurrenten**: Kennst du 1-3 Wettbewerber-Domains?
> 5. **GSC**: Hast du Google Search Console für diese Domain eingerichtet?
>
> Ich kann auch mit Minimalinfos starten und den Rest während der Arbeit herausfinden."

Erstelle dann eine Site-Config mit den gegebenen Infos. Felder die nicht bekannt sind mit `null` füllen – sie werden bei Bedarf ergänzt.

**Template**: Siehe `/config/SITE-CONFIG.md`

---

## Schritt 3: An Orchestrator/Modul übergeben

Lies die SKILL.md des gewählten Orchestrators/Moduls und folge dessen Ablauf.

**Wichtig**: Lies IMMER die SKILL.md des Moduls bevor du es ausführst. Die Module enthalten API-spezifische Hinweise, Feldnamen und Fehlerbehandlung die du brauchst.

---

## Entscheidungsbaum (Zusammenfassung)

```
User-Anfrage
    │
    ├── Neuer Artikel?
    │   ├── Site-Config vorhanden? → Orchestrator: New Article
    │   └── Keine Config? → Config erstellen → Orchestrator: New Article
    │
    ├── Artikel überarbeiten?
    │   ├── Site-Config vorhanden? → Orchestrator: Revise Article
    │   └── Keine Config? → Config erstellen → Orchestrator: Revise Article
    │
    ├── Einzelner Schritt?
    │   └── Direkt zum passenden Modul
    │
    └── Unklar?
        └── Frage nach: "Soll ich einen neuen Artikel erstellen oder einen bestehenden überarbeiten?"
```

---

## Dateien-Übersicht

```
seo-blog-pipeline/
├── config/
│   ├── SITE-CONFIG.md              ← Template + Anleitung
│   ├── style-guide.md              ← Stil-Regeln (für alle Sites)
│   ├── blacklist-words.md          ← Verbotene Wörter (für alle Sites)
│   └── sites/
│       └── [domain-slug].json      ← Pro-Website-Konfiguration
│
├── orchestrators/
│   ├── new-article/SKILL.md        ← Pfad A: Neuer Artikel
│   └── revise-article/SKILL.md     ← Pfad B: Artikel überarbeiten
│
└── modules/
    ├── competitor-research/SKILL.md ← Modul 1
    ├── seo-outline/SKILL.md         ← Modul 2
    ├── article-writer/SKILL.md      ← Modul 3
    ├── quality-check/SKILL.md       ← Modul 4
    ├── image-generation/SKILL.md    ← Modul 5
    └── internal-linking/SKILL.md    ← Modul 6
```

---

## Input/Output-Kette (Gesamtfluss)

```
Modul 1: Competitor Research
  Output: competitor-research-results.json
    ↓
Modul 2: SEO Outline
  Input:  competitor-research-results.json (oder Keyword direkt)
  Output: outline.json
    ↓
Modul 3: Article Writer
  Input:  outline.json + style-guide.md + blacklist-words.md
  Output: article.md + article-meta.json
    ↓                          ↑
Modul 4: Quality Check    ←───┘ (Feedback-Loop, max 2x)
  Input:  article.md + article-meta.json + outline.json
  Output: quality-report.json + revision-notes.md (bei REVISION_NEEDED)
    ↓
Modul 5: Image Generation
  Input:  article.md (APPROVED) + outline.json (image_suggestions)
  Output: article-with-images.md + image-manifest.json + PNG-Dateien
    ↓
Modul 6: Internal Linking
  Input:  article-with-images.md + outline.json + Site-Config
  Output: article-linked.md (= FINALER Artikel) + link-report.json
```
