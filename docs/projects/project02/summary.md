# Dokumentation — Projekt 2 (Voice Cloning)

## Architektur

Dieses Projekt untersucht verschiedene Ansätze für Voice Cloning mit generativer KI, speziell für die deutsche Sprache.

```text
┌─────────────────────────────────────────────────────────────────┐
│                    VOICE CLONING PIPELINE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. DATENVORBEREITUNG                                           │
│  ┌──────────────┐    VAD + Slice    ┌──────────────┐           │
│  │  Video (MP4) │ ─────────────────▶│  WAV-Segmente│           │
│  │  mit Sprecher│    5-15 Sek.     │  (3-10 Sek.) │           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
│  2. ZERO-SHOT TTS                                                 │
│  ┌──────────────┐    Referenz +   ┌──────────────┐           │
│  │  Segmente    │ ── Text ────────▶│  Synthetische│           │
│  │  (Stimme)    │                  │  Sprache     │           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
│  3. BEWERTUNG                                                   │
│  ┌──────────────┐                                             │
│  │  Text-Klarheit│  Stimmen-Ähnlichkeit  Sound-Qualität      │
│  │  Räumlichkeit│                                             │
│  └──────────────┘                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Test-Strategie

### Dimensions-Bewertung

Alle Tests werden nach vier Kriterien bewertet (je 1-5 Sterne):

| Kriterium | Beschreibung | Gewichtung |
|-----------|---------------|------------|
| **Text-Klarheit** | Wie gut ist der Text verständlich? | Hoch |
| **Stimmen-Ähnlichkeit** | Klingt es wie die Original-Stimme? | Hoch |
| **Sound-Qualität** | Klangqualität ohne Artefakte? | Mittel |
| **Räumlichkeit** | Natürlicher Raumklang/Stereo? | Mittel |

### Test-Texts

```text
Standard-Test (für alle Modelle):
"Hallo, das ist ein Test. Wie klingt diese Stimme?"

Langer Text (Drift-Test):
"Hallo und herzlich willkommen. Dies ist ein Test der deutschen 
Sprachausgabe. Ich hoffe es klingt natürlich und klar."
```

## Ergebnis-Zusammenfassung

| Phase | Modell | Hardware | Gesamt | Status |
|-------|--------|----------|--------|--------|
| Phase 1 | F5-TTS | CPU (Docker) | 2,5/5 | ✅ Erste Erkenntnisse |
| Phase 2 | GPT-SoVITS v2 | GPU (RTX A6000) | 0/5 | ❌ Training gescheitert |
| Phase 3 | XTTS v2 | GPU (RTX A6000) | 2,75/5 | ⚠️ Bestes Ergebnis |
| Phase 4 | CosyVoice v2 | GPU (RTX A6000) | 2/5 | ⚠️ Guter Klang, kein Deutsch |

## Zentrale Erkenntnis

!!! warning "Kritische Einsicht"
    **Kein Open-Source-Modell erreicht Produktionsqualität für deutsches Voice-Cloning.**
    
    Die verfügbaren Modelle (XTTS v2, CosyVoice, GPT-SoVITS) sind primär auf Englisch und Chinesisch trainiert. Deutsch wird toleriert, aber nicht meistert.
    
    **Lösungsansätze:**
    1. **Mehr Trainingsdaten:** 30+ Minuten deutsche Audio + Fine-Tuning
    2. **Kommerzielle APIs:** ElevenLabs, Play.ht (kostenpflichtig)
    3. **Hybrid-Ansatz:** Mehrere Modelle kombinieren

## Ressourcen

### Verwendete Modelle

| Modell | URL | Lizenz |
|--------|-----|--------|
| F5-TTS | https://github.com/SWivid/F5-TTS | MIT |
| GPT-SoVITS v2 | https://github.com/RVC-Boss/GPT-SoVITS | MIT |
| XTTS v2 | https://github.com/coqui-ai/TTS | CPML |
| CosyVoice v2 | https://github.com/FunAudioLLM/CosyVoice | Apache 2.0 |

### Tools

| Tool | Verwendung |
|------|-----------|
| FFmpeg | Audio-Extraktion, Format-Konvertierung |
| PyAnnote | Voice Activity Detection (VAD) |
| Whisper | ASR (Automatic Speech Recognition) |
| ModelScope | Chinesischer Model-Hub |
| tmux | Persistente Terminal-Sessions |

### Hardware-Anforderungen

| Komponente | Minimum | Empfohlen |
|------------|---------|-----------|
| GPU | — | NVIDIA RTX A6000 (48 GB) |
| RAM | 8 GB | 32 GB |
| Storage | 50 GB | 500 GB (SSD) |
| CUDA | — | 12.1+ |

## Weiterführende Links

- [Projekt 2 Übersicht](../projects/project02/index.md)
- [F5-TTS Phase](projects/project02/f5tts.md)
- [GPT-SoVITS Training](projects/project02/gptsovits.md)
- [XTTS v2 Tests](projects/project02/xtts.md)
- [CosyVoice Tests](projects/project02/cosyvoice.md)
- [Audio-Beispiele](projects/project02/audio.md)
- [Technische Details](projects/project02/technical.md)
