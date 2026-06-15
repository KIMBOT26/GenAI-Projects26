# Technische Details — Installation & Konfiguration

## Hardware-Setup

### Dual-VM Architektur

```text
┌─────────────────────────────────────────────────────────────────┐
│                        INFRASTRUKTUR                            │
├──────────────────────────┬──────────────────────────────────────┤
│      CPU-VM (Docker)      │         GPU-VM (RTX A6000)          │
│                           │                                      │
│  ┌──────────────────┐    │    ┌──────────────────────────────┐  │
│  │  Hermes Agent     │────┼────│  Conda Environments:        │  │
│  │  (Audio Prep)     │    │    │  • gptsovits_storage        │  │
│  └──────────────────┘    │    │  • cosyvoice_env            │  │
│           │               │    │    (PyTorch 2.3.1 + CUDA)   │  │
│           ▼               │    └──────────────────────────────┘  │
│  ┌──────────────────┐    │    ┌──────────────────────────────┐  │
│  │  /home/lecture/   │◄───┼────│  /mnt/storage (500 GB)      │  │
│  │  voice_data/      │    │    │  • gptsovits_training/      │  │
│  │  voice_sync/      │    │    │  • cosyvoice/               │  │
│  └──────────────────┘    │    │  • modelscope_cache/        │  │
│                          │    │  • tmp/                     │  │
└──────────────────────────┴──────────────────────────────────────┘

Netzwerk: GPU-VM nur über Host erreichbar (10.150.24.23)
```

## Sync-Scripts

### sync_to_gpu.sh

```bash
#!/bin/bash
# Kopiert Trainingsdaten von CPU-VM zu GPU-VM

GPU_VM="lecture@10.150.24.23"
GPU_DIR="/mnt/storage/gptsovits_training/jaleth_voice"

rsync -avz --progress \
  /home/lecture/voice_data/audio/ \
  ${GPU_VM}:${GPU_DIR}/audio/

rsync -avz --progress \
  /home/lecture/voice_data/configs/ \
  ${GPU_VM}:${GPU_DIR}/configs/

echo "✅ Sync zu GPU-VM abgeschlossen"
```

### sync_from_gpu.sh

```bash
#!/bin/bash
# Holt Trainingsergebnisse von GPU-VM

GPU_VM="lecture@10.150.24.23"
GPU_DIR="/mnt/storage/gptsovits_training/jaleth_voice"

rsync -avz --progress \
  ${GPU_VM}:${GPU_DIR}/logs/ \
  /home/lecture/voice_data/output/

rsync -avz --progress \
  ${GPU_VM}:${GPU_DIR}/*.wav \
  /home/lecture/voice_data/output/

echo "✅ Sync von GPU-VM abgeschlossen"
```

## GPT-SoVITS v2 — Kritische Configs

### s2_custom.json (8 Pflichtfelder)

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

!!! danger "Crash bei Epoch 10"
    Fehlt eines dieser 8 Felder → Training crasht bei Epoch 10.

### s1_custom.yaml (Optimizer-Block)

```yaml
# OHNE diesen Block startet das Training NICHT
optimizer:
  lr: 5e-4
  betas: [0.9, 0.99]
  eps: 1e-9

# Weitere Parameter
train:
  seed: 42
  epochs: 100
  batch_size: 4
```

### 3-get-semantic.py Patch

```python
# VORHER (crasht):
get_semantic_fn = get_semantic_module(
    config_path=config_path,
    model_path=model_path,
    version=version  # ← MUSS entfernt werden
)

# NACHHER (funktioniert):
get_semantic_fn = get_semantic_module(
    config_path=config_path,
    model_path=model_path
)
```

## CosyVoice — Installations-Fixes

### ModelScope Cache Umleitung

```bash
# Problem: ModelScope speichert auf Systemdisk (nur 29 GB)
# Lösung: Symlink auf /mnt/storage (500 GB)

rm -rf ~/.cache/modelscope
mkdir -p /mnt/storage/modelscope_cache
ln -s /mnt/storage/modelscope_cache ~/.cache/modelscope
```

### YAML-Symlink (CosyVoice2-0.5B)

```bash
# Problem: cosyvoice.yaml wird erwartet, aber cosyvoice2.yaml geliefert
cd /mnt/storage/modelscope_cache/hub/iic/CosyVoice2-0___5B/
ln -s cosyvoice2.yaml cosyvoice.yaml
```

### whisper-Installation (Build-Fehler)

```bash
# Standard-Installation schlägt fehl:
pip install openai-whisper==20231117  # ❌ Build fehlschlägt

# Lösung: Ohne Build-Isolation
pip install --no-build-isolation openai-whisper==20231117  # ✅
```

## XTTS v2 — Inferenz

### TOS automatisch akzeptieren

```python
import TTS.utils.manage as manage

# Patch: TOS automatisch akzeptieren (für automatisierte Pipelines)
manage.ModelManager.ask_tos = lambda self, model_dir: True

from TTS.api import TTS

tts = TTS("tts_models/multilingual/multi-dataset/xtts_v2").to("cuda")

wav = tts.tts(
    text="Hallo, das ist ein Test.",
    speaker_wav="referenz.wav",
    language="de"
)
```

### Segment-Lösung (gegen Drift)

```python
import numpy as np
import soundfile as sf

texts = [
    "Hallo und herzlich willkommen.",
    "Dies ist ein Test der Sprachausgabe.",
    "Ich hoffe es klingt natürlich und klar."
]

all_audio = []
for text in texts:
    wav = tts.tts(text=text, speaker_wav=ref, language="de")
    all_audio.append(wav)

# Mit kurzen Pausen (0,3 Sek. = ~7200 Samples bei 24kHz)
pause = np.zeros(int(0.3 * 24000))
combined = []
for i, wav in enumerate(all_audio):
    combined.append(wav)
    if i < len(all_audio) - 1:
        combined.append(pause)

final = np.concatenate(combined)
sf.write("output_segmented.wav", final, 24000)
```

## Troubleshooting

| Problem | Ursache | Lösung |
|---------|---------|--------|
| `Out of Memory` | Batch-Size zu groß | `batch_size=1` setzen |
| `No module named 'matcha'` | Matcha-TTS nicht im PYTHONPATH | `sys.path.insert(0, '/pfad/zu/Matcha-TTS')` |
| `CUDA out of memory` | Andere Prozesse auf GPU | `nvidia-smi` → Prozesse killen |
| `ModuleNotFoundError: pyarrow` | Fehlende Abhängigkeit | `pip install pyarrow` |
| Training crasht Epoch 10 | `s2_custom.json` unvollständig | Alle 8 Felder prüfen |
| Training startet nicht | `s1_custom.yaml` ohne Optimizer | Optimizer-Block hinzufügen |

## Zentrale Dateien & Pfade

```text
/home/lecture/voice_data/
├── audio/              # Trainings-Audio (geschnitten)
├── configs/            # GPT-SoVITS Configs
│   ├── s1_custom.yaml
│   └── s2_custom.json
├── output/             # Generierte Audio-Dateien
└── voice_sync/         # Sync-Scripts
    ├── sync_to_gpu.sh
    └── sync_from_gpu.sh

/mnt/storage/gptsovits_training/jaleth_voice/
├── 4-cnhubert/         # HuBERT-Features
├── 5-wav32k/           # Audio-Segmente (279 Stück)
├── logs/               # Trainings-Logs & Checkpoints
├── configs/            # Kopie von voice_data/configs
└── *.wav               # Generierte Test-Audios

/mnt/storage/cosyvoice/
├── cosyvoice/          # CosyVoice Source Code
├── third_party/        # Matcha-TTS
│   └── Matcha-TTS/
└── *.py                # Test-Scripts

/mnt/storage/cosyvoice_env/     # Conda-Umgebung (PyTorch 2.3.1)
/mnt/storage/modelscope_cache/  # Heruntergeladene Modelle
```
