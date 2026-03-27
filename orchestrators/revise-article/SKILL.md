---
name: orchestrator-revise-article
description: Steuert die Überarbeitung eines bestehenden Blogartikels. Analysiert die aktuelle Performance, identifiziert Lücken und Verbesserungspotenzial, und überarbeitet den Artikel gezielt.
---

# Orchestrator: Artikel überarbeiten

## Wann diesen Orchestrator verwenden
Wenn ein **bestehender** Blogartikel verbessert werden soll. Typische Trigger:
- "Überarbeite unseren Artikel über [Thema]"
- "Dieser Artikel rankt schlecht, kannst du ihn verbessern?"
- "Aktualisiere den Ratgeber zu [Thema]"
- "[URL] braucht ein Update"
- "Mach eine Content-Gap-Analyse für [URL]"

## Benötigte Inputs vom User
- **Pflicht**: URL des bestehenden Artikels ODER Artikel als Markdown/Text
- **Pflicht**: Ziel-Domain (z.B. "example-shop.example")
- **Optional**: Konkretes Ziel (z.B. "mehr Traffic", "besseres Ranking für Keyword X")
- **Optional**: Bekannte Probleme (z.B. "der Artikel ist veraltet")

---

## Ablauf-Übersicht

```
User-Auftrag + bestehender Artikel
    │
    ▼
┌─────────────────────────┐
│ 0. Site-Config laden     │
│    + Artikel einlesen    │
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 1. Performance-Analyse   │  ← GSC + Ahrefs
│    + Gap-Analyse         │
└─────────┬───────────────┘
          │
     User bestätigt Strategie
          │
          ▼
┌─────────────────────────┐
│ 2. Revision-Outline      │  ← Modul 2 (Revision-Modus)
└─────────┬───────────────┘
          │
     User prüft Outline
          │
          ▼
┌─────────────────────────┐
│ 3. Article Writer        │  ← Modul 3 (Revision-Modus)
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 4. Quality Check         │  ← Modul 4 (mit Feedback-Loop)
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 5. Image Generation      │  ← Modul 5 (nur neue Bilder)
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 6. Internal Linking      │  ← Modul 6 (validiert alle Links)
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 7. Finales Output-Paket  │
│    + Diff zum Original   │
└─────────────────────────┘
```

---

## Schritt-für-Schritt

### Schritt 0: Site-Config laden + Artikel einlesen

**Site-Config**:
```
Pfad: /config/sites/[domain-slug].json
```

**Bestehenden Artikel laden**:
- Wenn URL gegeben: `web_fetch` um den Artikel zu laden
- Wenn Datei gegeben: Direkt einlesen
- Extrahiere: H1, alle H2s/H3s, Wortanzahl, bestehende Links, Bilder

**Speichere als**: `original-article.md`

---

### Schritt 1: Performance-Analyse + Gap-Analyse

Dieser Schritt ersetzt Modul 1 (Competitor Research) für Revisionen. Er analysiert wie der bestehende Artikel performt und wo die Lücken sind.

#### 1a: GSC Performance holen

```
Tool: gsc:search_analytics
Parameter:
  siteUrl: [seo.gsc_property aus Site-Config]
  startDate: [90 Tage zurück]
  endDate: [heute]
  dimensions: query
  pageFilter: [URL des Artikels]
  filterOperator: contains
  rowLimit: 50
```

**Analysiere**:
- Für welche Keywords rankt der Artikel bereits?
- Welche haben hohes Volumen aber niedrige CTR? (→ Title/Meta verbessern)
- Welche ranken auf Position 4-20? (→ Content verbessern kann diese nach oben bringen)
- Gibt es Keywords die auf Seite 2 stehen? (→ gezielte Optimierung lohnt sich)

#### 1b: Ahrefs-Daten holen

```
Tool: ahref:site-explorer-organic-keywords
Parameter:
  target: [URL des Artikels OHNE https://]
  date: [heute]
  country: [site.country_code aus Site-Config]
  mode: exact
  limit: 30
  select: keyword,volume,best_position,sum_traffic,difficulty
```

#### 1c: Konkurrenz für das Haupt-Keyword analysieren

```
Tool: ahref:serp-overview
Parameter:
  keyword: [Haupt-Keyword des Artikels]
  country: [country_code]
  select: url,title,position,traffic,keywords,backlinks
  top_positions: 10
```

**Analysiere**:
- Wer rankt über uns?
- Was haben die, was wir nicht haben? (Inhalt, Backlinks, Struktur)
- Welche H2-Themen decken Konkurrenten ab, die wir nicht haben?

#### 1d: Gap-Analysis erstellen

Erstelle `gap-analysis.json`:

```json
{
  "meta": {
    "date": "YYYY-MM-DD",
    "article_url": "[URL]",
    "current_main_keyword": "[Keyword]",
    "current_position": 12,
    "current_traffic": 45
  },
  "performance": {
    "total_keywords": 28,
    "keywords_top3": 2,
    "keywords_top10": 8,
    "keywords_page2": 12,
    "impressions_90d": 5400,
    "clicks_90d": 180,
    "avg_ctr": "3.3%"
  },
  "quick_wins": [
    {
      "keyword": "gartenmöbel reinigen hausmittel",
      "current_position": 8,
      "volume": 480,
      "action": "Keyword stärker in H2 und Absatz einbauen"
    }
  ],
  "content_gaps": [
    {
      "topic": "Dampfreiniger für Gartenmöbel",
      "evidence": "Top-3 Konkurrenten haben alle einen Abschnitt dazu, wir nicht",
      "potential_keywords": ["dampfreiniger gartenmöbel", "gartenmöbel dampfreinigen"],
      "action": "Neuen H2-Abschnitt hinzufügen"
    }
  ],
  "structural_issues": [
    {
      "issue": "H1 enthält nicht das Haupt-Keyword",
      "current": "So werden deine Stühle wieder sauber",
      "suggested": "Gartenmöbel reinigen: So werden sie wieder frisch"
    }
  ],
  "competitor_advantages": [
    {
      "competitor": "example-competitor.example",
      "advantage": "FAQ-Sektion mit 8 Fragen – wir haben keine",
      "action": "FAQ-Abschnitt mit 5-7 relevanten Fragen ergänzen"
    }
  ],
  "revision_priority": [
    "1. H1 und Meta-Description optimieren (Quick Win)",
    "2. Fehlende H2-Abschnitte ergänzen (Dampfreiniger, FAQ)",
    "3. Bestehende Abschnitte mit Keywords anreichern",
    "4. Interne Verlinkung prüfen und erweitern"
  ]
}
```

**User-Entscheidung erforderlich**:
> "Ich habe deinen Artikel analysiert. Hier sind die Ergebnisse:
>
> **Aktuelle Performance**: Rankt für [N] Keywords, davon [N] auf Seite 1
> **Haupt-Keyword**: '[Keyword]' auf Position [X]
>
> **Verbesserungspotenzial**:
> 1. [Quick Win 1]
> 2. [Content Gap 1]
> 3. [Structural Issue 1]
>
> **Empfohlene Strategie**: [Zusammenfassung]
>
> Soll ich den Artikel entsprechend überarbeiten?"

**Warte auf User-Antwort.**

---

### Schritt 2: Revision-Outline (Modul 2 – Revision-Modus)

**Ausführung**:
- Lies SKILL.md: `/modules/seo-outline/SKILL.md` → Abschnitt "Spezial-Modus: Revision-Outline"
- Input: `gap-analysis.json` + `original-article.md`
- Erstelle Outline mit `"action"` pro H2: `keep | revise | new | merge | remove`

**Output**: `outline.json` (mit Revision-Markierungen)

**User-Entscheidung erforderlich**:
> "Hier ist der Überarbeitungsplan:
>
> **Beibehalten**: [H2] ✓, [H2] ✓
> **Überarbeiten**: [H2] – [Was sich ändert]
> **Neu hinzufügen**: [H2] – [Warum nötig]
> **Entfernen/Zusammenführen**: [H2] – [Grund]
>
> **Geschätzte neue Wortanzahl**: [X] (vorher: [Y])
>
> Passt das so?"

---

### Schritt 3: Article Writer – Revision-Modus (Modul 3)

**Ausführung**:
- Lies SKILL.md: `/modules/article-writer/SKILL.md` → Schritt 4 (Revision-Modus)
- Input: `outline.json` + `original-article.md`
- Nur Abschnitte ändern die mit `revise` oder `new` markiert sind
- `keep`-Abschnitte NICHT anfassen
- `remove`-Abschnitte entfernen
- Neue `[INTERNER_LINK: ...]` Platzhalter setzen wo nötig

**Output**: `article.md` + `article-meta.json` (mit `revision_iteration: 0`)

---

### Schritt 4: Quality Check + Feedback-Loop (Modul 4)

Identisch zum New-Article Orchestrator:
- Lies SKILL.md: `/modules/quality-check/SKILL.md`
- Max 2 Revision-Loops
- Score ≥ 90 oder nach 2 Loops → APPROVED

---

### Schritt 5: Image Generation (Modul 5)

**Nur für NEUE Abschnitte**:
- Bestehende Bilder behalten (aus original-article.md)
- Nur für neu hinzugefügte Abschnitte Bilder generieren
- Bestehende Bild-URLs nicht ersetzen

---

### Schritt 6: Internal Linking (Modul 6)

**Alle Links prüfen – auch bestehende**:
- Bestehende Links aus dem Original-Artikel: Validieren (404-Check)
- Neue `[INTERNER_LINK: ...]` Platzhalter: Zu echten URLs auflösen
- Ggf. zusätzliche relevante Links ergänzen

---

### Schritt 7: Finales Output-Paket

```
Revision-Paket: "[H1 des Artikels]"
├── article-revised.md         ← Überarbeiteter Artikel
├── original-article.md        ← Original zum Vergleich
├── gap-analysis.json          ← Performance-Analyse
├── outline.json               ← Revision-Outline
├── quality-report.json        ← Quality-Check Ergebnis
├── link-report.json           ← Link-Validierung
├── image-manifest.json        ← Neue Bilder
└── revision-summary.md        ← Zusammenfassung aller Änderungen
```

**revision-summary.md** enthält:
```markdown
# Revision Summary: [Artikeltitel]

## Änderungen im Überblick
- **Wortanzahl**: [vorher] → [nachher] (+X Wörter)
- **H2-Abschnitte**: [vorher] → [nachher] (+X neue)
- **Keywords abgedeckt**: [vorher] → [nachher]
- **Interne Links**: [vorher] → [nachher]
- **Quality Score**: [X]/100

## Was wurde geändert
1. **H1 optimiert**: "[alt]" → "[neu]"
2. **Neuer Abschnitt**: "[H2]" – [Kurzbeschreibung]
3. **Überarbeitet**: "[H2]" – [Was geändert wurde]
4. **Links**: [N] neue Links, [N] bestehende validiert, [N] defekte entfernt

## Erwarteter Impact
- Haupt-Keyword "[X]": Position [Y] → erwartet Top [Z]
- Neue Keywords abgedeckt: [Liste]
- Geschätzter Traffic-Zuwachs: +[X]% (basierend auf SERP-Daten)
```

---

## Fehlerbehandlung

| Situation | Aktion |
|-----------|--------|
| Artikel-URL nicht erreichbar | User nach alternativem Zugang fragen |
| GSC nicht verfügbar für diese Domain | Nur Ahrefs-Daten verwenden |
| Artikel hat kein klares Haupt-Keyword | Aus GSC/Ahrefs das stärkste Keyword identifizieren |
| Sehr kurzer Artikel (<300 Wörter) | Eher wie "New Article" behandeln, Original als Inspiration |
| Artikel bereits gut (Score >90 nach Analyse) | User informieren, nur kleinere Optimierungen vorschlagen |

---

## Unterschiede zum New-Article Orchestrator

| Aspekt | New Article | Revise Article |
|--------|------------|----------------|
| Schritt 1 | Competitor Research (Modul 1) | Performance + Gap-Analyse |
| Outline | Komplett neu | Revision-Markierungen (keep/revise/new) |
| Schreiben | Von Grund auf | Nur markierte Abschnitte ändern |
| Bilder | Alle neu | Nur neue Abschnitte |
| Links | Alle neu als Platzhalter | Bestehende validieren + neue auflösen |
| Output | Artikel-Paket | Revision-Paket mit Diff + Summary |
