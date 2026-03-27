---
name: image-generation
description: Generiert passende Bilder für einen Blogartikel basierend auf den Bild-Platzhaltern und Beschreibungen aus dem Outline. Nutzt Gemini Image Generation und optional Image Search für Referenzbilder.
---

# Modul 5: Image Generation

## Wann diesen Skill verwenden
Nach Modul 4 (Quality Check) wenn der Artikel APPROVED ist, oder parallel zum Quality Check. Ersetzt die BILD_PLATZHALTER im Artikel durch echte Bilder.

## Inputs
- **Pflicht**: `article.md` (APPROVED Artikel mit BILD_PLATZHALTER_X)
- **Pflicht**: `outline.json` (enthält image_suggestions mit Beschreibungen und Alt-Texten)
- **Optional**: Referenzbilder vom User oder aus Image Search

## Output
- Generierte Bilder als PNG-Dateien
- `article-with-images.md` – Artikel mit aktualisierten Bildpfaden (BILD_PLATZHALTER → echte Dateinamen)
- `image-manifest.json` – Übersicht aller generierten Bilder

---

## Ablauf

### Schritt 1: Bild-Anforderungen sammeln

Lies `outline.json` → `image_suggestions` und matche mit den BILD_PLATZHALTER im Artikel.

Für jedes Bild extrahiere:
- Platzhalter-ID (BILD_PLATZHALTER_1, _2, etc.)
- Beschreibung (was soll zu sehen sein)
- Alt-Text (SEO-optimiert, wird beibehalten)
- Platzierung im Artikel (nach welcher H2)
- Typ: Foto, Infografik, Vergleichsbild, Diagramm

### Schritt 2: Bild-Typ bestimmen

Nicht jedes Bild sollte generiert werden. Entscheide pro Bild:

| Bild-Typ | Methode | Beispiel |
|-----------|---------|---------|
| Produktfoto | SKIP – vom User bereitstellen lassen | Echte Teakholz-Nahaufnahme |
| Infografik | AI-Generierung | Flecktyp → Hausmittel Übersicht |
| Vergleichsbild | AI-Generierung | Vorher/Nachher, Alt vs. Neu |
| Schema/Diagramm | AI-Generierung | Raumskizze mit Anordnung |
| How-To Illustration | AI-Generierung | Schritt-für-Schritt Visualisierung |
| Stimmungsbild | AI-Generierung | Gemütliche saubere Terrasse |

**Regel**: Wenn das Bild ein echtes Produkt der Ziel-Website zeigen soll → NICHT generieren, sondern als "USER_REQUIRED" markieren.

### Schritt 3: Prompts erstellen

Für jedes zu generierende Bild einen detaillierten Prompt schreiben:

**Prompt-Struktur**:
```
[Bildtyp]: [Hauptmotiv]
Stil: [Fotorealistisch / Illustration / Flat Design / Infografik]
Perspektive: [Vogelperspektive / Augenhöhe / Nahaufnahme / Draufsicht]
Farben: [Farbpalette passend zum Branding]
Text im Bild: [Falls nötig, z.B. bei Infografiken – NUR auf Deutsch]
Hintergrund: [Weiß / Neutral / Wohnraum-Setting]
NICHT enthalten: [Was vermieden werden soll]
```

**Prompt-Regeln**:
- Immer auf Englisch prompten (bessere Ergebnisse)
- Deutscher Text im Bild nur wenn unbedingt nötig (KI-Text oft fehlerhaft)
- Kein Text in Fotos – nur in Infografiken wenn unvermeidbar
- Keine Gesichter von Personen
- Keine erkennbaren Markenlogos
- Clean, professionell, modern
- Passend zum Thema des Artikels

### Schritt 4: Bilder generieren

```
Tool: nano-banana-pro:generate_image
Parameter:
  prompt: [detaillierter englischer Prompt]
  aspectRatio: "16:9"  ← Standard für Blog-Bilder
  imageSize: "2K"
  outputPath: /home/claude/seo-blog-pipeline/output/images/[dateiname].png
```

**Aspect Ratios je nach Verwendung**:
- Blog-Header / Hauptbild: 16:9
- Inline-Bild: 4:3
- Infografik / Vergleich: 3:4 oder 1:1
- Pinterest-optimiert: 9:16

**Dateinamen-Konvention**:
`[artikel-slug]-[bild-nummer]-[kurzbeschreibung].png`
Beispiel: `gartenmoebel-reinigen-01-hausmittel-natron.png`

### Schritt 5: Qualitätsprüfung

Für jedes generierte Bild:

```
Tool: nano-banana-pro:describe_image
Parameter:
  images: [generiertes Bild]
  prompt: "Describe this image. Is it suitable for a German blog article about [Thema]? Check for: text errors, inappropriate content, brand logos, quality issues."
```

**Prüfe**:
- [ ] Bild passt zum Kontext des Artikelabschnitts
- [ ] Kein fehlerhafter Text im Bild
- [ ] Keine unpassenden Elemente
- [ ] Ausreichende Qualität / nicht verschwommen
- [ ] Keine Wasserzeichen oder Artefakte

Bei Nicht-Bestehen: Prompt anpassen und neu generieren (max 2 Versuche pro Bild).

### Schritt 6: Bilder in Artikel einsetzen

Ersetze im `article.md` die Platzhalter:

Vorher:
```markdown
![Alt-Text](BILD_PLATZHALTER_1)
```

Nachher:
```markdown
![Gartenmöbel reinigen mit Natron – Hausmittel Anwendung](gartenmoebel-reinigen-01-hausmittel-natron.png)
```

**Wichtig**: Alt-Texte aus dem Outline beibehalten – die sind SEO-optimiert.

### Schritt 7: Output generieren

**Bilder**: Alle PNG-Dateien in `/output/images/`

**article-with-images.md**: Artikel mit eingesetzten Bildpfaden

**image-manifest.json**:
```json
{
  "meta": {
    "created_date": "2026-02-19",
    "article": "gartenmoebel-reinigen",
    "total_images": 4
  },
  "images": [
    {
      "placeholder": "BILD_PLATZHALTER_1",
      "filename": "gartenmoebel-reinigen-01-hausmittel-natron.png",
      "alt_text": "Gartenmöbel reinigen mit Natron – Hausmittel Anwendung",
      "aspect_ratio": "4:3",
      "placement": "Nach H2 'Gartenmöbel reinigen mit Hausmitteln'",
      "method": "ai_generated",
      "prompt_used": "Close-up photo of white baking soda powder sprinkled on a weathered wooden garden table, with a wooden spoon nearby. Clean, bright, minimalist patio setting. No text, no people.",
      "quality_check": "passed"
    },
    {
      "placeholder": "BILD_PLATZHALTER_2",
      "filename": "gartenmoebel-reinigen-02-flecken-infografik.png",
      "alt_text": "Übersicht: Welches Hausmittel gegen welchen Fleck auf Gartenmöbeln",
      "aspect_ratio": "1:1",
      "placement": "Im Abschnitt 'Flecken entfernen'",
      "method": "ai_generated",
      "prompt_used": "Clean infographic showing stain types and remedies...",
      "quality_check": "passed"
    },
    {
      "placeholder": "BILD_PLATZHALTER_3",
      "filename": "gartenmoebel-reinigen-03-materialien.png",
      "alt_text": "Verschiedene Gartenmöbel-Materialien: Teakholz, Aluminium, Polyrattan",
      "aspect_ratio": "16:9",
      "placement": "Im Abschnitt 'Materialien richtig pflegen'",
      "method": "user_required",
      "note": "Echte Produktfotos der Ziel-Website empfohlen",
      "quality_check": "skipped"
    },
    {
      "placeholder": "BILD_PLATZHALTER_4",
      "filename": "gartenmoebel-reinigen-04-vorher-nachher.png",
      "alt_text": "Vergleich: vergrauter Gartenstuhl neben gereinigtem Gartenstuhl",
      "aspect_ratio": "16:9",
      "placement": "Im Abschnitt 'Wann austauschen'",
      "method": "user_required",
      "note": "Echtes Vergleichsfoto empfohlen (altes vs. neues Produkt)",
      "quality_check": "skipped"
    }
  ]
}
```

---

## Spezialfälle

### Infografiken mit deutschem Text
KI-generierte Bilder mit deutschem Text sind oft fehlerhaft. Zwei Optionen:
1. **Bevorzugt**: Bild ohne Text generieren, Text später im CMS/Canva hinzufügen
2. **Alternativ**: Bild mit englischem Text generieren, ins Deutsche übersetzen lassen

### Produktfotos
Niemals echte Produktfotos der Ziel-Website mit AI generieren. Diese als `"method": "user_required"` markieren mit Hinweis welches Produkt fotografiert werden soll.

### Referenzbilder nutzen
Wenn der User Referenzbilder bereitstellt (Stil, Farbgebung):
```
Tool: nano-banana-pro:generate_image
Parameter:
  prompt: [Prompt]
  images: [Referenzbild als base64]
```

---

## API-Kosten
- ~1 Gemini-Aufruf pro Bild (generate_image)
- ~1 Gemini-Aufruf pro Qualitätsprüfung (describe_image)
- Bei Neugeneration: +2 Aufrufe pro gescheitertem Bild
- Typisch: 3-6 generate + 3-6 describe = 6-12 Aufrufe pro Artikel

---

## Nächster Schritt
`article-with-images.md` wird an **Modul 6: Internal Linking** übergeben, das die `[INTERNER_LINK: ...]` Platzhalter durch echte, validierte URLs ersetzt.
