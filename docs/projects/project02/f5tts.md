# Phase 1 — F5-TTS Erstversuch

## Überblick

**Modell:** [F5-TTS](https://github.com/SWivid/F5-TTS) ("Less is More")  
**Ansatz:** Zero-Shot Voice Cloning auf CPU  
**Datum:** Erste Tests in einer früheren Session

## Warum F5-TTS?

F5-TTS war das erste Modell, das wir getestet haben, da es:
- Auf der CPU-VM laufen kann (keine GPU nötig)
- "Less is More"-Philosophie: weniger Postprocessing = besseres Ergebnis
- Einfach zu bedienen ist

## Architektur

```text
┌─────────────────────────────────────────────────────────────┐
│                    F5-TTS PIPELINE                           │
│                                                              │
│   REFERENZ-AUDIO          TEXT                  OUTPUT        │
│   ┌──────────────┐   ┌──────────────┐    ┌──────────────┐  │
│   │  3-10 Sek.   │   │  "Hallo..."  │───▶│  WAV-Datei   │  │
│   │  Sprechprobe  │   │              │    │  24 kHz      │  │
│   └──────────────┘   └──────────────┘    └──────────────┘  │
│                                                              │
│   Vorteile:                                                  │
│   ✅ Keine GPU nötig                                         │
│   ✅ Einfache Bedienung                                      │
│   ❌ Klingt "roboterhaft" / wie Niederländer                 │
└─────────────────────────────────────────────────────────────┘
```

## Erste Ergebnisse

Die ersten Tests mit F5-TTS ergaben folgende Charakteristiken:

### Positiv
- ✅ **Verständlichkeit:** Der generierte Text war verständlich
- ✅ **Konsistenz:** Keine plötzlichen Qualitätseinbrüche
- ✅ **Einfachheit:** Keine komplexe Konfiguration nötig

### Negativ
- ❌ **Stimmenqualität:** Klang wie "ein Niederländer, der Deutsch versucht"
- ❌ **Roboterhaft:** Deutlich künstlich, nicht natürlich
- ❌ **Fehlende Härte:** Deutsche Konsonanten zu weich

## Wichtige Erkenntnis

!!! tip "Less is More"
    F5-TTS zeigte: **Postprocessing und Parameter-Tweaks verschlechtern das Ergebnis.** Pure Defaults + Voice Activity Detection (VAD) war die beste Strategie. Jedes Experimentieren mit zusätzlichen Filtern oder Pitch-Shifting machte das Ergebnis nur schlechter.

## Bewertung

| Kriterium | Bewertung | Kommentar |
|-----------|-----------|-----------|
| **Text-Klarheit** | 3/5 | Verständlich, aber nicht perfekt |
| **Stimmen-Ähnlichkeit** | 2/5 | Kaum erkennbar |
| **Sound-Qualität** | 2,5/5 | OK für Prototypen |
| **Räumlichkeit** | 2/5 | Flach, kein Raumgefühl |
| **Gesamt** | **2,5/5** | Proof-of-Concept, keine Produktionsqualität |

## Fazit

F5-TTS war ein guter **Proof-of-Concept**, der zeigte, dass das grundsätzliche Prinzip funktioniert. Allerdings reichte die Qualität nicht für ernsthafte Anwendungen. Die Erkenntnis, dass weniger Postprocessing besser ist, war wertvoll für alle weiteren Experimente.

---

**Nächster Schritt:** [Phase 2: GPT-SoVITS Training](gptsovits.md)
