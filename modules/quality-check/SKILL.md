---
name: quality-check
description: Prüft einen fertigen Artikel gegen das Outline, den Style Guide, die Blacklist und SEO-Kriterien. Gibt entweder APPROVED oder REVISION_NEEDED mit konkreten Änderungsanweisungen zurück. Maximal 2 Revision-Loops.
---

# Modul 4: Quality Check

## Wann diesen Skill verwenden
Nach Modul 3 (Article Writer). Prüft den Artikel systematisch und entscheidet ob er veröffentlicht werden kann oder zurück an Modul 3 muss.

## Inputs
- **Pflicht**: `article.md` (Output von Modul 3)
- **Pflicht**: `article-meta.json` (Output von Modul 3)
- **Pflicht**: `outline.json` (Output von Modul 2)
- **Pflicht**: Style Guide (`/config/style-guide.md`)
- **Pflicht**: Blacklist (`/config/blacklist-words.md`)

## Output
- `quality-report.json` – Bewertung mit Score und Details
- Bei REVISION_NEEDED zusätzlich: `revision-notes.md` – Konkrete Änderungsanweisungen für Modul 3

---

## Ablauf

### Schritt 1: Config laden
Lies zuerst:
1. Site-Config (`/config/sites/[domain].json`) – für Pfade zu Style Guide und Blacklist, und Ton (du/Sie)
2. Style Guide (Pfad aus Site-Config `style.style_guide`)
3. Blacklist (Pfad aus Site-Config `style.blacklist`)

### Schritt 2: Prüfungen durchführen

Führe ALLE folgenden Prüfungen durch, in genau dieser Reihenfolge:

---

#### Prüfung 1: Struktur-Check (Outline-Treue)

Vergleiche `article.md` mit `outline.json`:

| Prüfpunkt | Kriterium | Schwere |
|-----------|-----------|---------|
| H1 korrekt | Exakt wie im Outline | BLOCKER |
| Alle H2s vorhanden | Jede H2 aus dem Outline existiert im Artikel | BLOCKER |
| H2-Reihenfolge | Gleiche Reihenfolge wie im Outline | WARNUNG |
| Wortanzahl gesamt | Innerhalb ±20% der Outline-Vorgabe | WARNUNG |
| Wortanzahl pro H2 | Innerhalb ±30% der Abschnitt-Vorgabe | WARNUNG |
| Meta-Description | Vorhanden, ≤155 Zeichen, enthält Haupt-Keyword | BLOCKER |
| Bild-Platzhalter | Alle im Outline definierten Bilder platziert | WARNUNG |

**Wie Wortanzahl messen**: Zähle die Wörter zwischen zwei H2-Überschriften. Nutze dafür eine einfache Wortzählung (Leerzeichen-getrennte Tokens).

---

#### Prüfung 2: Blacklist-Check

Durchsuche den gesamten Artikel nach verbotenen Wörtern und Phrasen aus `/config/blacklist-words.md`:

**Verbotene Wörter** (einzeln suchen, case-insensitive):
- nahtlos, reibungslos, ganzheitlich, maßgeschneidert, revolutionär, innovativ, einzigartig, streamlined

**Verbotene Phrasen** (als Teilstring suchen):
- "In der heutigen Zeit"
- "Im Zeitalter von"
- "Es versteht sich von selbst"
- "Zusammenfassend lässt sich festhalten"
- "Es ist kein Geheimnis"
- "In diesem Artikel erfährst du"
- "Abschließend lässt sich sagen"

**Verbotene Zeichen**:
- Em-Dash (—) → ersetzen durch Gedankenstrich (–) oder Satzumstellung
- Mehr als 1 Ausrufezeichen im gesamten Artikel

| Prüfpunkt | Schwere |
|-----------|---------|
| Verbotenes Wort gefunden | BLOCKER |
| Verbotene Phrase gefunden | BLOCKER |
| Em-Dash gefunden | BLOCKER |
| >1 Ausrufezeichen | WARNUNG |

---

#### Prüfung 3: Stil-Check

Prüfe jeden Absatz gegen die Regeln aus `/config/style-guide.md`:

| Prüfpunkt | Kriterium | Schwere |
|-----------|-----------|---------|
| Absatzlänge | ≤4 Sätze pro Absatz | BLOCKER |
| Satzlänge | Keine Sätze mit >2 Nebensätzen | WARNUNG |
| Du/Sie-Form | Konsequent gemäß Site-Config `style.tone` (kein Mix) | BLOCKER |
| Passiv-Anteil | <20% der Sätze in Passiv | WARNUNG |
| Fettdruck | ≤3 fettgedruckte Elemente pro Abschnitt | WARNUNG |

**Wie Absatzlänge messen**: Ein Absatz endet bei einer Leerzeile. Zähle die Sätze (Punkt, Frage- oder Ausrufezeichen als Satzende). Überschriften, Listenpunkte und Bild-Platzhalter zählen nicht als Sätze.

**Wie Nebensätze zählen**: Zähle Kommas die einen Nebensatz einleiten (dass, weil, wenn, obwohl, während, da, etc.). Aufzählungs-Kommas (A, B und C) zählen nicht.

---

#### Prüfung 4: SEO-Check

| Prüfpunkt | Kriterium | Schwere |
|-----------|-----------|---------|
| Haupt-Keyword in H1 | Muss enthalten sein | BLOCKER |
| Haupt-Keyword in Einleitung | In den ersten 100 Wörtern | BLOCKER |
| Haupt-Keyword Dichte | 0,5–2,5% (bezogen auf Gesamtwortanzahl) | WARNUNG |
| Neben-Keywords | Mind. 60% der Neben-Keywords aus outline.json verwendet | WARNUNG |
| LSI-Terms | Mind. 40% der LSI-Terms aus outline.json verwendet | WARNUNG |
| Interne Links | Mind. 2 Link-Platzhalter vorhanden `[INTERNER_LINK: ...]` | WARNUNG |
| Interne Links Format | Platzhalter-Format korrekt: `[INTERNER_LINK: Ankertext \| Thema]` | WARNUNG |
| Alt-Texte | Alle Bild-Platzhalter haben Alt-Texte | WARNUNG |
| H2-Keywords | Mind. 50% der H2s enthalten ein Neben-Keyword | WARNUNG |

**Wie Keyword-Dichte berechnen**:
Anzahl_Keyword / Gesamtwortanzahl × 100 = Prozent
Bei Mehrwort-Keywords: Suche nach der exakten Phrase.

---

#### Prüfung 5: Inhalt-Check

| Prüfpunkt | Kriterium | Schwere |
|-----------|-----------|---------|
| Kein Intro-Verräter | Kein "In diesem Artikel...", "Im Folgenden..." | BLOCKER |
| Kein Schluss-Verräter | Kein "Zusammenfassend...", "Abschließend..." | BLOCKER |
| Einleitung Länge | 50–100 Wörter | WARNUNG |
| FAQ-Format | FAQ-Fragen als H3, Antworten ≤3 Sätze | WARNUNG |
| Faktische Plausibilität | Keine offensichtlich falschen Behauptungen | BLOCKER |
| Content Gap adressiert | Mind. 1 Content Gap aus outline.json behandelt | WARNUNG |

**Faktische Plausibilität**: Prüfe Zahlen, Zeitangaben und Mengenangaben auf Plausibilität. Wenn eine Behauptung verdächtig klingt:

```
Tool: web_search
Query: [faktische Behauptung verifizieren]
```

Markiere nur als BLOCKER wenn die Behauptung nachweislich falsch ist. Bei Unsicherheit als WARNUNG markieren mit dem Hinweis "Faktencheck empfohlen".

---

### Schritt 3: Score berechnen

**Scoring-System**:
- Starte bei 100 Punkten
- Pro BLOCKER: -15 Punkte
- Pro WARNUNG: -5 Punkte
- Minimum: 0 Punkte

**Bewertung**:
- 90-100: APPROVED – Artikel kann veröffentlicht werden
- 70-89: REVISION_NEEDED – Kleine Korrekturen nötig
- <70: REVISION_NEEDED – Größere Überarbeitung nötig

---

### Schritt 4: Entscheidung

#### Bei APPROVED (Score ≥90, keine BLOCKER):
- Erstelle `quality-report.json` mit Score und allen Prüfergebnissen
- Artikel geht weiter an Modul 5 (Image Generation) oder direkt zur Veröffentlichung

#### Bei REVISION_NEEDED:
- Erstelle `quality-report.json` mit Score und allen Prüfergebnissen
- Erstelle `revision-notes.md` mit konkreten Änderungsanweisungen
- Artikel geht zurück an Modul 3

---

### Schritt 5: Revision Notes erstellen (nur bei REVISION_NEEDED)

Die `revision-notes.md` muss für Modul 3 eindeutig verständlich sein.

Format für jedes Problem:

```markdown
## Problem [Nummer]: [Kurzbeschreibung]

**Schwere**: BLOCKER / WARNUNG
**Prüfung**: [Name der Prüfung]
**Stelle im Artikel**: [Zitat oder H2-Abschnitt]
**Problem**: [Was ist falsch]
**Lösung**: [Was Modul 3 konkret tun soll]
```

Beispiel:

```markdown
## Problem 1: Verbotenes Wort "einzigartig"

**Schwere**: BLOCKER
**Prüfung**: Blacklist-Check
**Stelle im Artikel**: Abschnitt "Materialien richtig pflegen", Satz: "Die einzigartige Maserung von Teakholz..."
**Problem**: "einzigartig" steht auf der Blacklist
**Lösung**: Ersetze "einzigartige" durch "besondere" oder "markante"

## Problem 2: Absatz zu lang (5 Sätze)

**Schwere**: BLOCKER
**Prüfung**: Stil-Check
**Stelle im Artikel**: Abschnitt "Hausmittel", 2. Absatz
**Problem**: Absatz hat 5 Sätze, Maximum sind 4
**Lösung**: Letzten Satz in einen neuen Absatz verschieben oder mit dem vorletzten Satz zusammenfassen

## Problem 3: Keyword-Dichte zu niedrig

**Schwere**: WARNUNG
**Prüfung**: SEO-Check
**Stelle im Artikel**: Gesamter Artikel
**Problem**: "gartenmöbel reinigen" kommt 8x vor (0,4%), Minimum ist 0,5%
**Lösung**: 3-4 weitere natürliche Erwähnungen einbauen, z.B. in den Abschnitten Dampfreiniger und FAQ
```

---

## Feedback-Loop Regeln

- **Maximum 2 Revision-Loops**: Wenn der Artikel nach 2 Korrekturdurchgängen immer noch BLOCKER hat → APPROVED mit Hinweis auf verbleibende Probleme
- **revision_iteration** in `article-meta.json` trackt den Loop-Zähler
- Bei Iteration 2+: Nur noch BLOCKER werden zurückgegeben, WARNUNGEN werden ignoriert
- Jede Iteration muss den Score verbessern – wenn nicht, ist etwas am Outline falsch

---

## Output-Format

### quality-report.json

```json
{
  "meta": {
    "checked_date": "2026-02-19",
    "article_file": "article-gartenmoebel-reinigen.md",
    "outline_file": "outline-gartenmoebel-reinigen.json",
    "revision_iteration": 0
  },
  "score": 65,
  "verdict": "REVISION_NEEDED",
  "summary": "Die Struktur stimmt, aber 1 Blacklist-Verstoß und 1 zu langer Absatz müssen korrigiert werden. Dazu kommt 1 SEO-Warnung.",
  "checks": {
    "structure": {
      "passed": true,
      "issues": []
    },
    "blacklist": {
      "passed": false,
      "issues": [
        {
          "severity": "BLOCKER",
          "word": "einzigartig",
          "location": "Abschnitt 'Materialien richtig pflegen', Satz 3",
          "suggestion": "Ersetze durch 'besondere' oder 'markante'"
        }
      ]
    },
    "style": {
      "passed": false,
      "issues": [
        {
          "severity": "BLOCKER",
          "type": "paragraph_too_long",
          "location": "Abschnitt 'Hausmittel', Absatz 2",
          "detail": "5 Sätze (max 4)",
          "suggestion": "Letzten Satz abtrennen"
        }
      ]
    },
    "seo": {
      "passed": true,
      "issues": [
        {
          "severity": "WARNUNG",
          "type": "keyword_density_low",
          "detail": "0,4% statt mind. 0,5%",
          "suggestion": "3-4 weitere natürliche Erwähnungen"
        }
      ]
    },
    "content": {
      "passed": true,
      "issues": []
    }
  },
  "stats": {
    "total_blockers": 2,
    "total_warnings": 1,
    "word_count": 1420,
    "keyword_density_percent": 0.4,
    "paragraph_count": 28,
    "avg_sentences_per_paragraph": 2.8,
    "internal_link_placeholders_count": 2,
    "image_placeholders_count": 3,
    "du_form_consistent": true
  }
}
```

---

## API-Kosten
- Hauptsächlich kostenlos (Text-Analyse)
- 0-3 Web Searches für Faktencheck (nur bei verdächtigen Behauptungen)
- Keine Ahrefs-Kosten

---

## Nächster Schritt
- Bei APPROVED: Weiter an **Modul 5: Image Generation**
- Bei REVISION_NEEDED: Zurück an **Modul 3: Article Writer** mit `revision-notes.md`
