# Phase 2 — GPT-SoVITS Training

## Überblick

**Modell:** [GPT-SoVITS v2](https://github.com/RVC-Boss/GPT-SoVITS)  
**Ansatz:** Vollständiges Fine-Tuning auf eigener Stimme (GPU)  
**Trainingsdaten:** ~3 Minuten deutsche Audio-Segmente

```text
┌──────────────────────────────────────────────────────────────────┐
│              GPT-SoVITS v2 — 5-Schritt Pipeline                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Schritt 1: Audio vorbereiten                                    │
│  ┌──────────────┐    VAD + Slice    ┌──────────────┐           │
│  │  Roh-Audio   │ ─────────────────▶ │  Segmente    │           │
│  │  (3 Min.)    │    (5-15 Sek.)   │  (279 Stück) │           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
│  Schritt 2: ASR (Automatic Speech Recognition)                   │
│  ┌──────────────┐    Whisper      ┌──────────────┐           │
│  │  Segmente    │ ────────────────▶ │  Textlabels  │           │
│  │  (WAV)       │    "large-v2"   │  (.list)     │           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
│  Schritt 3: HuBERT-Features extrahieren                        │
│  ┌──────────────┐    cnhubert     ┌──────────────┐           │
│  │  Segmente    │ ────────────────▶ │  .pt-Files   │           │
│  │  (WAV)       │                  │  (Embeddings)│           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
│  Schritt 4: Semantische Tokens (s1_train)                      │
│  ┌──────────────┐    Transformer   ┌──────────────┐           │
│  │  Text +      │ ────────────────▶ │  s1 Modell   │           │
│  │  Audio       │    ~100 Epochen  │  (ca. 3h)    │           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
│  Schritt 5: VAE-Decoder (s2_train)                             │
│  ┌──────────────┐    GAN-Training  ┌──────────────┐           │
│  │  Semantik +  │ ────────────────▶ │  s2 Modell   │           │
│  │  Audio       │    ~100 Epochen  │  (ca. 6h)    │           │
│  └──────────────┘                  └──────────────┘           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Dual-VM Workflow

```text
CPU-VM (Hermes Agent)                          GPU-VM (RTX A6000)
┌─────────────────────────┐                   ┌─────────────────────────┐
│  1. Audio vorbereiten   │                   │                         │
│  2. Segmente erstellen  │                   │                         │
│  3. Dateien sammeln     │ ──sync_to_gpu──▶  │  4. Training starten   │
│                         │                   │  5. s1_train.py        │
│                         │                   │  6. s2_train.py        │
│                         │ ◀─sync_from_gpu─  │  7. Checkpoints holen  │
└─────────────────────────┘                   └─────────────────────────┘

Sync-Scripts:
• sync_to_gpu.sh   → Kopiert audio/ + configs/ zur GPU-VM
• sync_from_gpu.sh → Holt logs/ + Modelle zurück
```

## Kritische Probleme

### Problem 1: ARPA-Deutsch statt IPA

GPT-SoVITS v2 verwendet für die Phonetisierung **ARPA** (amerikanisches Englisch) statt **IPA** (Internationales Phonetisches Alphabet). Das führt zu:

- "ch" wird wie englisches "tsch" gesprochen
- Umlaute (ä, ö, ü) werden falsch interpretiert
- Deutsche Konsonantenkluster fehlen

### Problem 2: s2_custom.json — Fehlende Felder

!!! danger "Crash bei Epoch 10"
    Die `s2_custom.json` braucht **8 Pflichtfelder**, sonst crasht das Training bei Epoch 10 mit einem kryptischen Fehler:

    ```json
    {
      "pretrained_s2G": "GPT_SoVITS/pretrained_models/s2G488k.pth",
      "pretrained_s2D": "GPT_SoVITS/pretrained_models/s2D488k.pth",
      "save_every_epoch": 4,
      "if_save_latest": true,
      "if_save_every_weights": true,
      "fp": 16,
      "version": "v2",
      "exp_dir": "logs/jaleth_voice"
    }
    ```

### Problem 3: s1_custom.yaml — Fehlender Optimizer-Block

Der Default-Config fehlt der **Optimizer-Block** im YAML:

```yaml
optimizer:
  lr: 5e-4
  betas: [0.9, 0.99]
  eps: 1e-9
```

Ohne diesen Block: Training startet nicht.

### Problem 4: 3-get-semantic.py Patch

Das Script hat einen Bug — `version=version` muss entfernt werden:

```python
# VORHER (crasht):
get_semantic_fn = get_semantic_module(..., version=version)

# NACHHER (funktioniert):
get_semantic_fn = get_semantic_module(...)
```

## Training-Ergebnis

Nach **12 Stunden Training** auf der RTX A6000:

```text
┌─────────────────────────────────────────────────────────┐
│           INFERENZ-TEST                                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Eingabe: "Hallo, das ist ein Test."                   │
│                                                         │
│  Output:  ████████████████████████████████████████     │
│          (reines weißes Rauschen — keine Sprache)      │
│                                                         │
│  Grund: Zu wenig Trainingsdaten (~3 Min. statt 30+)   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

!!! failure "Ergebnis"
    **Reines Rauschen.** 3 Minuten deutsche Audio reichen nicht für GPT-SoVITS Fine-Tuning. Das Modell hat die Sprachmerkmale nie gelernt.

## Fehleranalyse

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Rauschen | Zu wenig Daten (< 30 Min.) | Mehr Audio sammeln |
| Deutsches ARPA | Fehlende deutsche Phonetik | IPA-basiertes Modell |
| Config-Crashes | Undokumentierte Pflichtfelder | Dokumentation lesen |

## Bewertung

| Kriterium | Bewertung | Kommentar |
|-----------|-----------|-----------|
| **Text-Klarheit** | 0/5 | Nur Rauschen |
| **Stimmen-Ähnlichkeit** | 0/5 | Keine Stimme erkennbar |
| **Sound-Qualität** | 0/5 | Reines Rauschen |
| **Räumlichkeit** | 0/5 | — |
| **Gesamt** | **0/5** | Vollständiger Fehlschlag |

## Lessons Learned

1. **Mehr Daten braucht man** — 3 Minuten ist für modernes TTS-Training zu wenig
2. **Deutsche Sprache ist ein Problem** — Die meisten Modelle sind Englisch/Chinesisch-zentriert
3. **Config-Fehler sind tödlich** — Kleine JSON/YAML-Fehler führen zu Stunden verschwendeter GPU-Zeit
4. **Tmux ist essentiell** — Für 12h-Training ohne Session-Verlust

---

**Nächster Schritt:** [Phase 3: XTTS v2 Tests](xtts.md)
