# Phase 4 — CosyVoice Tests

## Überblick

**Modell:** [CosyVoice v2](https://github.com/FunAudioLLM/CosyVoice) (Alibaba)  
**Ansatz:** Zero-Shot Voice Cloning mit chinesischem Multilingual-Modell  
**Hardware:** NVIDIA RTX A6000 (48 GB VRAM)

CosyVoice ist ein von Alibaba entwickeltes Text-to-Speech Modell, das stark auf Chinesisch und Englisch trainiert wurde. Die Hoffnung: Vielleicht beherrscht es Deutsch besser als XTTS v2?

## Installation

Die Installation war komplex — separate Conda-Umgebung nötig:

```bash
# Neue Conda-Umgebung auf /mnt/storage (nicht Systemdisk!)
conda create -p /mnt/storage/cosyvoice_env python=3.10 -y
conda activate /mnt/storage/cosyvoice_env

# PyTorch 2.3.1 (CosyVoice braucht genau diese Version)
pip install torch==2.3.1 torchaudio==2.3.1 \
  --index-url https://download.pytorch.org/whl/cu121

# CosyVoice-spezifische Pakete
pip install conformer==0.3.2 diffusers==0.29.0 \
  x-transformers==2.11.24 wetext==0.0.4

# Matcha-TTS (Abhängigkeit)
git clone https://github.com/shivammehta25/Matcha-TTS.git \
  /mnt/storage/cosyvoice/third_party/Matcha-TTS
```

## Architektur

```text
┌───────────────────────────────────────────────────────────────────┐
│                    COSYVOICE PIPELINE                             │
│                                                                   │
│   REFERENZ (6 Sek.)          TEXT                     OUTPUT      │
│   ┌───────────────┐      ┌───────────────┐       ┌───────────┐ │
│   │ WAV-Datei     │      │  Deutscher    │       │ WAV       │ │
│   │ (Stimme)      │ ────▶│  Text          │ ────▶│ 22.05kHz │ │
│   └───────────────┘      └───────────────┘       └───────────┘ │
│                                                                   │
│   Modell: iic/CosyVoice-300M (via ModelScope Hub)                │
│   Frontend: wetext (Text-Normalisierung)                         │
│   Sprache: Keine deutsche Phonetik vorhanden                     │
└───────────────────────────────────────────────────────────────────┘
```

## Installation-Probleme

### Problem 1: ModelScope Cache auf Systemdisk

```
# FEHLER: ModelScope speichert Modelle unter ~/.cache/ (Systemdisk)
# LÖSUNG: Symlink auf /mnt/storage
rm -rf ~/.cache/modelscope
mkdir -p /mnt/storage/modelscope_cache
ln -s /mnt/storage/modelscope_cache ~/.cache/modelscope
```

### Problem 2: YAML-Konflikt

```
# FEHLER: cosyvoice.yaml vs. cosyvoice2.yaml
# CosyVoice2-0.5B erwartet "cosyvoice.yaml", liefert aber "cosyvoice2.yaml"
# LÖSUNG: Manueller Symlink
cd /mnt/storage/modelscope_cache/hub/iic/CosyVoice2-0___5B/
ln -s cosyvoice2.yaml cosyvoice.yaml
```

### Problem 3: Fehlende Abhängigkeiten

```python
# pyarrow fehlte → ModuleNotFoundError
pip install pyarrow

# onnxruntime fehlte → ONNX-Modelle konnten nicht geladen werden
pip install onnxruntime==1.18.0

# whisper fehlte → Build fehlgeschlagen
pip install --no-build-isolation openai-whisper==20231117
```

## Test-Ergebnis

```python
import sys
sys.path.insert(0, '/mnt/storage/cosyvoice')
sys.path.insert(0, '/mnt/storage/cosyvoice/third_party/Matcha-TTS')

from cosyvoice.cli.cosyvoice import CosyVoice

cosyvoice = CosyVoice('iic/CosyVoice-300M')

# Zero-Shot Test
for result in cosyvoice.inference_zero_shot(
    "Hallo und herzlich willkommen.",
    "Hallo, das ist ein Test.",
    "referenz.wav"
):
    audio = result['tts_speech']
```

**Generierungs-Zeit:** ~2-5 Sekunden pro Satz  
**Sample Rate:** 22.050 Hz

### Audio-Analyse

| Aspekt | Bewertung |
|--------|-----------|
| **Klang** | 3,5/5 — Guter, klarer Sound |
| **Aussprache** | 0,5/5 — Nur das Wort "Test" verständlich |
| **Deutsch** | Fehlende deutsche Phonetik |
| **Stabilität** | Kein Drift bei kurzen Sätzen |

## Warum CosyVoice bei Deutsch versagt

```text
┌─────────────────────────────────────────────────────────────────┐
│           COSYVOICE TRAININGSSPRACHEN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Chinesisch (Mandarin)  ████████████████████████████████  60% │
│   Englisch               ██████████████                    25% │
│   Japanisch              ████                              8%  │
│   Deutsch                ▌                                 0%  │
│                                                                 │
│   CosyVoice hat KEINE deutschen Trainingsdaten!               │
│   Die wetext-Frontend kann Text normalisieren,               │
│   aber das neuronale Netzwerk kennt keine deutschen Laute.  │
│                                                                 │
│   Ergebnis: Guter Klang, aber Unverständlich                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Bewertung

| Kriterium | Bewertung | Kommentar |
|-----------|-----------|-----------|
| **Text-Klarheit** | 0,5/5 | Fast nichts verständlich |
| **Stimmen-Ähnlichkeit** | 1/5 | Keine erkennbare Stimme |
| **Sound-Qualität** | 3,5/5 | Guter Klang, wenn man nicht auf Text achtet |
| **Räumlichkeit** | 2/5 | Flach |
| **Gesamt** | **2/5** | Nicht nutzbar für Deutsch |

## Vergleich mit XTTS v2

| Modell | Klang | Aussprache | Deutsch | Gesamt |
|--------|-------|------------|---------|--------|
| **XTTS v2** | 3,5/5 | 3,5/5 | ⚠️ Quengelig | 2,75/5 |
| **CosyVoice** | 3,5/5 | 0,5/5 | ❌ Keine | 2/5 |

**Ergebnis:** CosyVoice hat zwar besseren "Rohklang", aber da es kein Deutsch kann, ist XTTS v2 für deutsche Anwendungen die bessere (wenn auch unzureichende) Wahl.

## Fazit

CosyVoice ist ein beeindruckendes chinesisches TTS-Modell mit guter Klangqualität. Aber **es kann kein Deutsch** — und das ist kein Konfigurationsproblem, sondern ein fundamentales Trainingsproblem.

!!! danger "Harte Wahrheit"
    **Es gibt aktuell kein Open-Source-Modell, das out-of-the-box hochwertiges deutsches Voice-Cloning kann.**

---

**Nächster Schritt:** [Audio-Beispiele](audio.md) — Alle getesteten Samples im direkten Vergleich
