---
name: article-writer
description: Schreibt einen vollständigen SEO-optimierten Blogartikel auf Deutsch (Du-Form) basierend auf einem Outline. Nutzt Web Search für Fakten-Recherche. Befolgt strikte Stil-Regeln aus dem zentralen Style Guide und der Blacklist.
---

# Modul 3: Article Writer

## Wann diesen Skill verwenden
Wenn ein Blogartikel geschrieben oder überarbeitet werden soll. Kommt nach Modul 2 (SEO Outline) und liefert an Modul 4 (Quality Check).

## Inputs

### Für neuen Artikel
- **Pflicht**: `outline.json` (Output von Modul 2)
- **Pflicht**: Style Guide (`/config/style-guide.md`)
- **Pflicht**: Blacklist (`/config/blacklist-words.md`)

### Für Überarbeitung (Revision-Modus)
- **Pflicht**: `outline.json` mit `"action": "keep|revise|new|merge|remove"` pro Abschnitt
- **Pflicht**: `original-article.md` (bestehender Artikel)
- **Optional**: `revision-notes.md` (vom Quality Check, bei Feedback-Loop)

## Output
- `article.md` – Fertiger Artikel in Markdown
- `article-meta.json` – Metadaten (Wortanzahl, Keywords verwendet, interne Links)

---

## Vor dem Schreiben: Config laden

Lies IMMER zuerst diese Dateien:
1. Site-Config (`/config/sites/[domain].json`) – Ton (du/Sie), Sprache, Style-Guide-Pfad
2. Style Guide (Pfad aus Site-Config `style.style_guide`) – Alle Schreibregeln
3. Blacklist (Pfad aus Site-Config `style.blacklist`) – Verbotene Wörter und Phrasen

Die Anrede (Du/Sie) wird aus `style.tone` der Site-Config gelesen. Diese Regeln sind NICHT verhandelbar.

---

## Ablauf

### Schritt 1: Outline analysieren

Lies `outline.json` und extrahiere:
- H1 und alle H2s
- Keyword-Map (Haupt-Keyword, Neben-Keywords, LSI-Terms)
- Briefing pro Abschnitt
- Wortanzahl-Vorgaben pro Abschnitt
- Vorgeschlagene interne Links
- Bild-Platzierungen

### Schritt 2: Fakten-Recherche

Für jeden H2-Abschnitt: Prüfe ob du aktuelle Fakten brauchst.

```
Tool: web_search
Query: [themenspezifische Suchanfrage auf Deutsch]
```

**Recherchiere für**:
- Konkrete Zahlen und Statistiken die im Briefing erwähnt werden
- Aktuelle Produktinformationen oder Preisbereiche
- Fachbegriffe die korrekt erklärt werden müssen
- Gesetzliche Vorgaben (Brandschutz, Normen, etc.)

**Recherchiere NICHT für**:
- Allgemeinwissen das du sicher kennst
- Meinungen oder subjektive Bewertungen
- Informationen die im Outline bereits als verifiziert markiert sind

**Regel**: Lieber eine Faktenbehauptung weglassen als eine falsche aufstellen. Wenn du unsicher bist, formuliere vorsichtiger ("oft", "in vielen Fällen") statt absolute Aussagen zu machen.

### Schritt 3: Artikel schreiben

Schreibe den Artikel Abschnitt für Abschnitt. Befolge dabei STRIKT die folgenden Regeln:

---

#### STIL-REGELN (nicht verhandelbar)

**Sprache & Ton**
- Sprache und Anrede gemäß Site-Config (`style.tone`): du-form, sie-form, oder neutral
- Professionell aber gesprächig
- Wie ein Experte der einem schlauen Freund etwas erklärt
- KEIN Pressemitteilungs-Ton
- KEIN Marketing-Sprech
- KEINE Übertreibungen

**Satzebene**
- Kurze, klare Sätze
- Maximal EIN Nebensatz pro Satz
- Aktiv statt Passiv wo möglich
- Verbal statt Nominal ("Miss nach" statt "Die Durchführung der Messung")

**Absatzebene**
- Maximal 3-4 Sätze pro Absatz
- Lieber zu kurz als zu lang
- Jeder Absatz hat EINEN Kerngedanken
- Zwischen Absätzen: logischer Übergang, aber kein "Darüber hinaus"

**Überschriften**
- H1: Genau wie im Outline vorgegeben
- H2: Genau wie im Outline vorgegeben (nicht eigenmächtig ändern!)
- H3: Nur wenn ein H2-Abschnitt wirklich Untergliederung braucht
- Keine H4 oder tiefer

**Verbotene Wörter und Phrasen**
Prüfe JEDEN Satz gegen `/config/blacklist-words.md`. Dazu gehören u.a.:
- nahtlos, reibungslos, ganzheitlich, maßgeschneidert, revolutionär, innovativ
- "In der heutigen Zeit", "Im Zeitalter von", "Es ist kein Geheimnis"
- Em-Dashes (—), übermäßige Ausrufezeichen
- Sätze mit mehr als 2 Nebensätzen

Wenn du ein verbotenes Wort verwenden willst → ersetze es SOFORT durch die Alternative aus der Blacklist.

**Formatierung**
- Fettdruck nur für wirklich wichtige Begriffe, nicht inflationär (max 2-3 pro Abschnitt)
- Listen/Aufzählungen nur wo sie echten Mehrwert bieten
- Keine verschachtelten Listen
- Keine Emojis (außer der User verlangt es explizit)

---

#### INHALTLICHE REGELN

**Einleitung (50-100 Wörter)**
- Sprich das Problem des Lesers direkt an
- Zeige dass du verstehst was er sucht
- Deute die Lösung an
- Haupt-Keyword in den ersten 2 Sätzen
- KEIN "In diesem Artikel erfährst du..." – einfach anfangen

Beispiel für guten Einstieg:
```
Deine Gartenmöbel sind vergraut, haben grüne Flecken und riechen muffig?
Dann wird es Zeit für eine gründliche Reinigung. Das Gute: In den meisten
Fällen brauchst du dafür keinen teuren Profi. Mit den richtigen Hausmitteln
und etwas Geduld bekommst du deine Gartenmöbel wieder frisch.
```

Beispiel für schlechten Einstieg:
```
In der heutigen Zeit verbringen wir einen Großteil unserer Freizeit auf der
Terrasse. Daher ist es nicht verwunderlich, dass Gartenmöbel einer nahtlosen
Reinigung bedürfen. In diesem umfassenden Ratgeber zeigen wir Ihnen...
```

**Hauptteil**
- Folge EXAKT der H2-Reihenfolge aus dem Outline
- Halte dich an die Wortanzahl-Vorgaben pro Abschnitt (±20%)
- Baue Keywords natürlich ein – kein Keyword-Stuffing
- Jeder Abschnitt beantwortet eine konkrete Frage des Lesers
- Nutze das Briefing aus dem Outline als inhaltliche Leitplanke
- Wenn das Briefing Tabellen oder Listen vorschlägt: setze sie um

**Interne Links**
- Setze **Platzhalter-Links** statt echte URLs – Modul 6 (Internal Linking) löst diese später auf
- Format: `[INTERNER_LINK: Ankertext | Thema]`
- Beispiel: `[INTERNER_LINK: Gartentische aus Teakholz | Gartentisch Kategorie]`
- Der Ankertext soll natürlich und keyword-haltig sein
- Max 3-5 Platzhalter pro Artikel
- KEINE echten URLs verwenden – die kennt nur Modul 6

**Bilder**
- An den im Outline vorgeschlagenen Stellen Bild-Platzhalter einfügen
- Format: `![Alt-Text](BILD_PLATZHALTER_1)`
- Nummeriere die Platzhalter (BILD_PLATZHALTER_1, _2, _3, etc.)
- Alt-Texte aus dem Outline übernehmen

**Fazit/Schluss (50-80 Wörter)**
- Kurz und handlungsorientiert
- Was soll der Leser jetzt tun?
- Kein "Zusammenfassend lässt sich festhalten..."
- Einfach das Wichtigste auf den Punkt bringen
- Optional: Überleitung zum Produkt/Angebot (subtil, nicht werblich)

**FAQ (wenn im Outline vorgesehen)**
- Exakt die Fragen aus dem Outline verwenden
- Jede Antwort: 2-3 Sätze, direkt und klar
- Strukturiert für FAQ-Schema:
```markdown
### Wie kann man Gartenmöbel selbst reinigen?
Am einfachsten mit Natron: Gleichmäßig aufstreuen, 30 Minuten einwirken
lassen, abspülen. Für hartnäckige Flecken hilft eine Mischung aus
Spülmittel und lauwarmem Wasser.
```

---

### Schritt 4: Revision-Modus (wenn revision-notes.md vorhanden)

Wenn der Artikel vom Quality Check zurückkommt:

1. Lies `revision-notes.md` vollständig
2. Gehe Problem für Problem durch
3. Für jedes Problem:
   - Finde die betroffene Stelle im Artikel
   - Führe die angeforderte Änderung durch
   - Achte darauf dass der umgebende Text noch flüssig klingt
4. Ändere NUR was in den Revision Notes steht
5. Verändere NICHT Abschnitte die nicht beanstandet wurden

**Wichtig**: Bei Halluzinations-Korrekturen:
- Entferne die falsche Information komplett
- Ersetze durch verifizierte Info (Web Search) ODER
- Formuliere allgemeiner wenn keine verifizierte Quelle verfügbar

---

### Schritt 5: Self-Check vor Übergabe

Bevor du den Artikel an Modul 4 übergibst, prüfe selbst:

- [ ] H1 exakt wie im Outline?
- [ ] Alle H2s aus dem Outline vorhanden und in richtiger Reihenfolge?
- [ ] Haupt-Keyword in Einleitung und natürlich verteilt?
- [ ] Alle Absätze ≤ 4 Sätze?
- [ ] Keine Sätze mit mehr als 2 Nebensätzen?
- [ ] Keine verbotenen Wörter aus der Blacklist?
- [ ] Keine Em-Dashes (—)?
- [ ] Konsequent Du-Form?
- [ ] Link-Platzhalter eingebaut? (`[INTERNER_LINK: ...]`)
- [ ] Bild-Platzhalter an den richtigen Stellen?
- [ ] Gesamtwortzahl innerhalb ±20% der Outline-Vorgabe?
- [ ] Kein "In diesem Artikel", "Zusammenfassend", "Abschließend"?

---

### Schritt 6: Output generieren

**article.md** – Der fertige Artikel:
```markdown
# [H1 aus Outline]

[Einleitung – 50-100 Wörter]

## [H2 aus Outline]

[Inhalt – gemäß Briefing und Wortanzahl]

![Alt-Text](BILD_PLATZHALTER_1)

## [Nächste H2]

[...]

## FAQ: [Titel aus Outline]

### [Frage 1]
[Antwort – 2-3 Sätze]

### [Frage 2]
[Antwort – 2-3 Sätze]
```

**article-meta.json** – Metadaten:
```json
{
  "title": "H1 des Artikels",
  "meta_description": "Aus Outline übernommen",
  "word_count": 1850,
  "primary_keyword": "gartenmöbel reinigen",
  "primary_keyword_count": 12,
  "secondary_keywords_used": ["terrassenmöbel reinigen", "hausmittel", "dampfreiniger"],
  "internal_links": [
    {"anchor": "Gartentische aus Teakholz", "topic": "Gartentisch Kategorie", "format": "placeholder"}
  ],
  "image_placeholders": 4,
  "h2_count": 8,
  "revision_iteration": 0,
  "blacklist_violations_found_and_fixed": 0
}
```

---

## API-Kosten
- Hauptsächlich Web Search für Fakten-Recherche
- Ca. 3-8 Web Search Aufrufe pro Artikel
- Keine Ahrefs-Kosten in diesem Modul

---

## Nächster Schritt
Output wird an **Modul 4: Quality Check** übergeben.
Bei Rückgabe durch Quality Check: `revision-notes.md` einlesen und Schritt 4 ausführen.
