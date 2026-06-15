# Projekt 2 — Voice Cloning mit generativer KI

## Zielsetzung

Entwicklung einer vollständig autonomen Pipeline zur Stimmenklonierung aus Video-Material. Die geklonte Stimme soll anschließend für Text-to-Speech (TTS) verwendet werden können.

```text
    VIDEO (mit Sprecher)                    GEKLONTE STIMME
    ┌─────────────────────┐                 ┌─────────────────────┐
    │  ┌───────────────┐  │   EXTRAKTION    │  ┌───────────────┐  │
    │  │   Audio-Track  │──┼─────────────────▶│   Referenz-WAV  │  │
    │  │   (Sprechen)   │  │   VAD + Slice   │  │  (3-10 Sek.)  │  │
    │  └───────────────┘  │                 │  └───────────────┘  │
    └─────────────────────┘                 │         │           │
                                            │         ▼           │
                                            │  ┌───────────────┐  │
                                            │  │  Zero-Shot    │  │
                                            │  │   TTS Modell  │──┼──▶ SYNTHETISCHE SPRACHE
                                            │  │               │  │
                                            │  └───────────────┘  │
                                            └─────────────────────┘
```

## Projektstruktur

| Phase | Modell | Status | Ergebnis |
|-------|--------|--------|----------|
| **Phase 1** | F5-TTS (CPU) | ✅ Erste Tests | Verständlich, aber "roboterhaft" |
| **Phase 2** | GPT-SoVITS v2 (GPU) | ❌ Training gescheitert | Nur Rauschen (zu wenig Daten) |
| **Phase 3** | XTTS v2 (GPU) | ⚠️ Teilerfolg | Verständlich, aber quengelig |
| **Phase 4** | CosyVoice v2 (GPU) | ⚠️ Teilerfolg | Guter Klang, aber keine deutsche Aussprache |

## Architektur

```text
┌─────────────────────────────────────────────────────────────────┐
│                        DUAL-VM SETUP                             │
├──────────────────────────┬──────────────────────────────────────┤
│      CPU-VM (Docker)      │         GPU-VM (RTX A6000)          │
│                           │                                      │
│  ┌──────────────────┐    │    ┌──────────────────────────────┐  │
│  │  Hermes Agent     │────┼────│  GPU-Training (CUDA 12.1)    │  │
│  │  (Audio Prep)     │    │    │  RTX A6000 (48 GB VRAM)      │  │
│  └──────────────────┘    │    └──────────────────────────────┘  │
│           │               │                  │                   │
│           ▼               │                  ▼                   │
│  ┌──────────────────┐    │    ┌──────────────────────────────┐  │
│  │  voice_data/      │    │    │  /mnt/storage (500 GB)       │  │
│  │  voice_sync/      │◄───┼────│  Modelle, Training, Output  │  │
│  └──────────────────┘    │    └──────────────────────────────┘  │
└──────────────────────────┴──────────────────────────────────────┘

           Sync via SSH: lecture@10.150.24.23
```

## Hardware-Spezifikationen

| Komponente | CPU-VM | GPU-VM |
|------------|--------|--------|
| **CPU** | x86_64 | x86_64 |
| **RAM** | 8 GB | 32 GB |
| **GPU** | — | NVIDIA RTX A6000 (48 GB VRAM) |
| **Disk** | 29 GB (System) | 29 GB (System) + 500 GB (/mnt/storage) |
| **OS** | Ubuntu 22.04 | Ubuntu 22.04 |

## Zentrale Erkenntnis

!!! warning "Kritische Einsicht"
    **Kein Open-Source-Modell macht out-of-the-box hochwertiges deutsches Voice-Cloning.** Die verfügbaren Modelle sind primär auf Englisch und Chinesisch trainiert. Für echte Qualität braucht man entweder:
    
    1. **30+ Minuten deutsche Trainingsdaten** + Fine-Tuning
    2. **Kommerzielle APIs** (z.B. ElevenLabs)

## Schnellnavigation

- [Phase 1: F5-TTS Erstversuch](f5tts.md) — Erste Experimente auf CPU
- [Phase 2: GPT-SoVITS Training](gptsovits.md) — Vollständiges Training auf GPU
- [Phase 3: XTTS v2 Tests](xtts.md) — Zero-Shot Versuche
- [Phase 4: CosyVoice Tests](cosyvoice.md) — Chinesisches Modell testen
- [Audio-Beispiele](audio.md) — Alle getesteten Samples mit Bewertungen
- [Technische Details](technical.md) — Installationsanleitungen & Konfigurationen
