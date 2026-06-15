# Audio-Beispiele — Alle Tests im Vergleich

## Referenz-Audio

Die Referenzstimme für alle Tests stammt aus einem deutschen Video. Extrahiert mittels Voice Activity Detection (VAD) und in 5-15 Sekunden Segmente geschnitten.

| Modell | Text | Dauer | Bewertung |
|--------|------|-------|-----------|
| **Referenz** | Originale Stimme aus dem Video | ~6 Sek. | Baseline |

## Phase 1: F5-TTS

### Erstversuch auf CPU

| Modell | Text | Dauer | Gesamt-Bewertung |
|--------|------|-------|------------------|
| **F5-TTS** | "Hallo, das ist ein Test" | ~3 Sek. | 2,5/5 |

**Charakteristik:**
- Klingt wie "ein Niederländer, der Deutsch versucht"
- Roboterhafte Artikulation
- Deutsche Härte (Konsonanten) fehlt komplett

## Phase 2: GPT-SoVITS

### Training-Ergebnis

| Modell | Text | Dauer | Gesamt-Bewertung |
|--------|------|-------|------------------|
| **GPT-SoVITS** | "Hallo, das ist ein Test" | ~3 Sek. | 0/5 |

**Charakteristik:**
- Reines weißes Rauschen
- Keine erkennbare Sprache
- Training mit nur 3 Min. Daten gescheitert

## Phase 3: XTTS v2

### Verschiedene Tests

| Test | Text | Dauer | Bewertung |
|------|------|-------|-----------|
| **Test 1** (Erste Generierung) | "Hallo, das ist ein Test" | ~3 Sek. | 2,5/5 |
| **Test 2** (Weniger Satzzeichen) | "Hallo das ist ein Test" | ~3 Sek. | 2,5/5 |
| **Test 3** (Schnelleres Tempo) | "Hallo, das ist ein Test" | ~2,5 Sek. | 2,75/5 |
| **Test 4** (Langer Text) | 3 Sätze zusammen | ~13 Sek. | 2/5 |
| **Test 5** (Kurze Segmente) | 3 Sätze einzeln | ~18 Sek. | 2,875/5 |

**Charakteristik:**
- Verständlich, aber quengelig/weich
- Bei langem Text: Wortsalat ab der Hälfte
- Mit Segment-Lösung: Stabil, aber quengelig bleibt

## Phase 4: CosyVoice

### Zero-Shot Test

| Modell | Text | Dauer | Gesamt-Bewertung |
|--------|------|-------|------------------|
| **CosyVoice-300M** | "Hallo und herzlich willkommen" | ~4 Sek. | 2/5 |

**Charakteristik:**
- Guter Klang (3,5/5)
- Aussprache katastrophal (0,5/5)
- Fast nichts verständlich — nur einzelne Wörter
- Modell hat kein Deutsch gelernt

## Vergleichstabelle

| Modell | Text-Klarheit | Stimmen-Ähnlichkeit | Sound-Qualität | Räumlichkeit | **Gesamt** |
|--------|--------------|---------------------|----------------|--------------|------------|
| **F5-TTS** (CPU) | 3/5 | 2/5 | 2,5/5 | 2/5 | **2,5/5** |
| **GPT-SoVITS** (GPU) | 0/5 | 0/5 | 0/5 | 0/5 | **0/5** |
| **XTTS v2** (GPU) | 3,5/5 | 2/5 | 3,5/5 | 2/5 | **2,75/5** |
| **CosyVoice** (GPU) | 0,5/5 | 1/5 | 3,5/5 | 2/5 | **2/5** |

## Fazit

```text
┌─────────────────────────────────────────────────────────────────┐
│              GESAMTERGEBNIS — DEUTSCHES VOICE CLONING           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Modell              Gesamt   Nutzbar für Deutsch?            │
│   ─────────────────────────────────────────────────────         │
│   F5-TTS              2,5/5    ⚠️ Prototypen (roboterhaft)      │
│   GPT-SoVITS          0/5      ❌ Nur Rauschen                  │
│   XTTS v2             2,75/5   ⚠️ Bestes Ergebnis, aber         │
│                                  quengelig und instabil         │
│   CosyVoice           2/5      ❌ Kein Deutsch                  │
│                                                                 │
│   ════════════════════════════════════════════════════════     │
│                                                                 │
│   KEIN Modell erreicht Produktionsqualität für Deutsch!        │
│                                                                 │
│   Empfohlene Alternativen:                                     │
│   1. Mehr Trainingsdaten (30+ Min.) + Fine-Tuning            │
│   2. Kommerzielle APIs (ElevenLabs, etc.)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```
