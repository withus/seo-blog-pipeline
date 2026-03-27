---
name: orchestrator-new-article
description: Steuert den kompletten Ablauf zur Erstellung eines neuen SEO-Blogartikels. Verkettet Module 1-6 in der richtigen Reihenfolge, handhabt User-Entscheidungen und den Quality-Check Feedback-Loop.
---

# Orchestrator: Neuer Artikel

## Wann diesen Orchestrator verwenden
Wenn ein **komplett neuer** Blogartikel erstellt werden soll. Typische Trigger:
- "Schreib einen neuen Blogartikel über [Thema]"
- "Erstelle Content für [Keyword]"
- "Analysiere die Konkurrenz und schreib einen Artikel"
- "Neuer Ratgeber-Artikel für [domain]"

## Benötigte Inputs vom User
- **Pflicht**: Ziel-Domain oder Website (z.B. "example-shop.example")
- **Optional**: Ziel-Keyword (z.B. "gartenmöbel reinigen")
- **Optional**: Konkurrenz-Domain(s) zur Analyse
- **Optional**: Thematische Vorgaben oder Fokus

---

## Ablauf-Übersicht

```
User-Auftrag
    │
    ▼
┌─────────────────────────┐
│ 0. Site-Config laden     │
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 1. Competitor Research   │  ← Modul 1
│    (falls kein Keyword)  │
└─────────┬───────────────┘
          │
     User wählt Keyword
          │
          ▼
┌─────────────────────────┐
│ 2. SEO Outline           │  ← Modul 2
└─────────┬───────────────┘
          │
     User prüft Outline
          │
          ▼
┌─────────────────────────┐
│ 3. Article Writer        │  ← Modul 3
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 4. Quality Check         │  ← Modul 4
│    Score ≥ 90? ──────────┼──→ ja ──→ weiter
│    Score < 90? ──────────┼──→ zurück zu Modul 3 (max 2x)
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 5. Image Generation      │  ← Modul 5
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 6. Internal Linking      │  ← Modul 6
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│ 7. Finales Output-Paket  │
└─────────────────────────┘
```

---

## Schritt-für-Schritt

### Schritt 0: Site-Config laden

Lies die Site-Config für die Ziel-Domain:
```
Pfad: /config/sites/[domain-slug].json
```

Wenn keine Site-Config existiert:
1. Frage den User nach den nötigsten Infos (Domain, Sprache, Themenbereich)
2. Erstelle eine minimale Site-Config
3. Informiere den User, dass bei Modul 6 die Sitemap gecrawlt wird

**Ab hier verwenden ALLE Module die Site-Config als zentrale Referenz.**

---

### Schritt 1: Competitor Research (Modul 1)

**Wann überspringen**: Wenn der User bereits ein konkretes Ziel-Keyword nennt UND keine Konkurrenz-Analyse möchte → direkt zu Schritt 2.

**Ausführung**:
- Lies SKILL.md: `/modules/competitor-research/SKILL.md`
- Verwende Konkurrenten aus Site-Config `competitors[]` oder vom User genannte Domains
- Ländercode aus Site-Config `site.country_code`

**Output**: `competitor-research-results.json`

**User-Entscheidung erforderlich**:
Präsentiere die Top-Empfehlungen und frage:
> "Ich habe die Konkurrenz analysiert. Hier sind die Top-Themen mit dem größten Potenzial:
> 1. [Thema A] – Volumen: X, Schwierigkeit: Y
> 2. [Thema B] – Volumen: X, Schwierigkeit: Y
> 3. [Thema C] – Volumen: X, Schwierigkeit: Y
>
> Für welches Thema soll ich das Outline erstellen?"

**Warte auf User-Antwort** bevor du weitergehst.

---

### Schritt 2: SEO Outline (Modul 2)

**Ausführung**:
- Lies SKILL.md: `/modules/seo-outline/SKILL.md`
- Verwende das gewählte Keyword + Site-Config
- GSC-Daten aus `seo.gsc_property` (falls verfügbar)

**Output**: `outline.json`

**User-Entscheidung erforderlich**:
Präsentiere das Outline übersichtlich:
> "Hier ist das Outline für '[Keyword]':
>
> **H1**: [Überschrift]
> **Geschätzte Wortanzahl**: [X] Wörter
>
> **Abschnitte**:
> 1. [H2] – [Kurzbeschreibung] (~X Wörter)
> 2. [H2] – [Kurzbeschreibung] (~X Wörter)
> ...
>
> **Content Gaps** die wir nutzen: [Auflistung]
>
> Soll ich so den Artikel schreiben, oder möchtest du etwas anpassen?"

**Warte auf User-Antwort** bevor du weitergehst. Wenn der User Änderungen will → Outline anpassen und nochmal zeigen.

---

### Schritt 3: Article Writer (Modul 3)

**Ausführung**:
- Lies SKILL.md: `/modules/article-writer/SKILL.md`
- Lade Style Guide und Blacklist (Pfade aus Site-Config)
- Schreibe den Artikel basierend auf dem genehmigten Outline
- Verwende `[INTERNER_LINK: Ankertext | Thema]` Platzhalter für interne Links

**Output**: `article.md` + `article-meta.json`

**Kein User-Stopp** – direkt weiter zum Quality Check.

---

### Schritt 4: Quality Check + Feedback-Loop (Modul 4)

**Ausführung**:
- Lies SKILL.md: `/modules/quality-check/SKILL.md`
- Prüfe den Artikel gegen Outline, Style Guide, Blacklist, SEO-Kriterien

**Output**: `quality-report.json` + ggf. `revision-notes.md`

#### Entscheidungslogik:

```
Score ≥ 90 UND keine BLOCKER?
  → APPROVED → weiter zu Schritt 5

Score < 90 ODER BLOCKER vorhanden?
  → REVISION_NEEDED
  → Prüfe: revision_iteration < 2?
      → ja: Zurück zu Modul 3 mit revision-notes.md
             → revision_iteration += 1
             → Modul 3 liest revision-notes.md und fixt nur die genannten Probleme
             → Zurück zu Modul 4 (erneuter Check)
      → nein (iteration ≥ 2): APPROVED mit Hinweisen
             → "Artikel nach 2 Revisionen freigegeben. Verbleibende Hinweise: [...]"
             → weiter zu Schritt 5
```

**User-Information** (kein Stopp):
> "Quality Check: Score [X]/100 – [APPROVED/REVISION_NEEDED]"
> Bei Revision: "Korrigiere [N] Probleme, neuer Durchgang..."
> Bei Freigabe: "Artikel freigegeben, generiere jetzt Bilder..."

---

### Schritt 5: Image Generation (Modul 5)

**Ausführung**:
- Lies SKILL.md: `/modules/image-generation/SKILL.md`
- Generiere Bilder für alle BILD_PLATZHALTER
- Markiere Produktfotos als USER_REQUIRED

**Output**: Bilder + `image-manifest.json` + `article-with-images.md`

**User-Information** bei USER_REQUIRED Bildern:
> "Für [N] Bilder empfehle ich echte Produktfotos:
> - BILD_PLATZHALTER_3: [Beschreibung welches Foto]
> - BILD_PLATZHALTER_4: [Beschreibung welches Foto]
>
> Diese kannst du später manuell ersetzen."

---

### Schritt 6: Internal Linking (Modul 6)

**Ausführung**:
- Lies SKILL.md: `/modules/internal-linking/SKILL.md`
- Lade Domain und URL-Muster aus Site-Config
- Crawle Sitemap (oder verwende `known_urls` aus Site-Config)
- Löse alle `[INTERNER_LINK: ...]` Platzhalter zu echten, validierten URLs auf
- Ergänze ggf. weitere passende Links

**Output**: `article-linked.md` + `link-report.json`

---

### Schritt 7: Finales Output-Paket

Stelle dem User das komplette Paket zusammen:

```
Artikel-Paket: "[H1 des Artikels]"
├── article-final.md          ← Fertiger Artikel mit echten Links + Bildpfaden
├── article-meta.json          ← Metadaten (Wörter, Keywords, Links)
├── outline.json               ← Das verwendete Outline
├── quality-report.json        ← Quality-Check Ergebnis
├── link-report.json           ← Alle validierten internen Links
├── image-manifest.json        ← Bild-Übersicht
└── images/                    ← Generierte Bilder
    ├── [slug]-01-[desc].png
    ├── [slug]-02-[desc].png
    └── ...
```

**Zusammenfassung für den User**:
> "Dein Artikel ist fertig! Hier die Zusammenfassung:
>
> **Titel**: [H1]
> **Wortanzahl**: [X] Wörter
> **Quality Score**: [X]/100
> **Interne Links**: [N] validierte Links gesetzt
> **Bilder**: [N] generiert, [N] vom User benötigt
> **Keyword-Abdeckung**: [X]% Neben-Keywords, [X]% LSI-Terms
>
> Was du noch tun musst:
> - [Ggf. Produktfotos bereitstellen]
> - [Ggf. Meta-Description in CMS eintragen]
> - Artikel ins CMS übertragen und veröffentlichen"

---

## Fehlerbehandlung

| Situation | Aktion |
|-----------|--------|
| Site-Config nicht gefunden | User fragen und minimale Config erstellen |
| Competitor Research liefert keine Ergebnisse | User nach alternativem Keyword fragen |
| Ahrefs API-Error | Fehler melden, mit verfügbaren Daten weiterarbeiten |
| GSC nicht verfügbar | Schritt überspringen, im Outline vermerken |
| Quality Check nach 2 Loops noch <70 | APPROVED mit Warnhinweis, User informieren |
| Sitemap nicht erreichbar | Web Search als Fallback für URL-Discovery |
| Image Generation fehlgeschlagen | Platzhalter belassen, User informieren |

---

## Geschätzte API-Kosten pro Artikel

| Modul | Ahrefs Units | Web Fetches | Web Searches | Image Gen |
|-------|-------------|-------------|-------------|-----------|
| M1 Competitor | ~1.500 | 0 | 0 | 0 |
| M2 Outline | ~500 | 3-5 | 0 | 0 |
| M3 Writer | 0 | 0 | 3-8 | 0 |
| M4 Quality | 0 | 0 | 0-3 | 0 |
| M5 Images | 0 | 0 | 0 | 6-12 |
| M6 Linking | 0 | 8-20 | 0-2 | 0 |
| **Gesamt** | **~2.000** | **11-25** | **3-13** | **6-12** |
