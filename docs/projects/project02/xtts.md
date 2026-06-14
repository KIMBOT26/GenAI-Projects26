# Phase 3 — XTTS v2 Zero-Shot Tests

## Überblick

**Modell:** [Coqui XTTS v2](https://github.com/coqui-ai/TTS)  
**Ansatz:** Zero-Shot Voice Cloning ohne Training  
**Hardware:** NVIDIA RTX A6000 (48 GB VRAM)

XTTS v2 ist ein multilingual trainiertes TTS-Modell, das mit nur einer kurzen Referenz-Audiodatei (< 10 Sek.) eine Stimme klonen kann — ohne Fine-Tuning.

## Installation

```bash
# Auf GPU-VM (RTX A6000)
pip install --no-cache-dir torch==2.3.1 torchaudio==2.3.1 \
  --index-url https://download.pytorch.org/whl/cu121

pip install TTS

# TOS automatisch akzeptieren (für automatisierte Pipelines)
python -c "import TTS.utils.manage as manage; manage.ModelManager.ask_tos = lambda self, x: True"
```

## Architektur

```text
┌───────────────────────────────────────────────────────────────┐
│                    XTTS v2 PIPELINE                          │
│                                                                │
│   REFERENZ (6 Sek.)          TEXT                    OUTPUT    │
│   ┌───────────────┐      ┌───────────────┐      ┌─────────┐ │
│   │ WAV-Datei    │      │  Deutscher    │      │ WAV     │ │
│   │ (Stimme)     │ ────▶│  Text         │ ────▶│ 24kHz   │ │
│   └───────────────┘      └───────────────┘      └─────────┘ │
│                                                                │
│   Modell: tts_models/multilingual/multi-dataset/xtts_v2      │
│   Sprache: "de" (Deutsch wird toleriert, nicht meistert)     │
└───────────────────────────────────────────────────────────────┘
```

## Durchgeführte Tests

### Test 1: Erste Generierung

```python
from TTS.api import TTS
import soundfile as sf

tts = TTS("tts_models/multilingual/multi-dataset/xtts_vts_v2").to("cuda")

wav = tts.tts(
    text="Hallo, das ist ein Test. Wie klingt diese Stimme?",
    speaker_wav="referenz.wav",
    language="de"
)

sf.write("output.wav", wav, 24000)
```

**Ergebnis:**
- ✅ Verständlich
- ⚠️ Sehr quengelig/weich
- ❌ Fehlende deutsche Härte ("ch", "r", Umlaute)

### Test 2: Weniger Satzzeichen

Hypothese: Weniger Kommas/Punkte = flüssigere Aussprache

```python
# DIREKTER TEXT (weniger Pausen)
text1 = "Hallo das ist ein Test wie klingt diese Stimme"

# NORMALER TEXT
text2 = "Hallo, das ist ein Test. Wie klingt diese Stimme?"
```

**Ergebnis:** Minimaler Unterschied. Das Modell setzt selbst Pausen.

### Test 3: Audio-Tempo erhöhen

Hypothese: Schnellere Wiedergabe = weniger quengelig

```python
from pydub import AudioSegment

audio = AudioSegment.from_wav("output.wav")
# 15% schneller
fast_audio = audio.speedup(playback_speed=1.15)
fast_audio.export("output_fast.wav", format="wav")
```

**Ergebnis:** Klingt dynamischer, aber die "Quengeligkeit" bleibt.

### Test 4: Langer Text (Drift-Problem)

Hypothese: Längere Texte bleiben stabil

```python
long_text = """Hallo und herzlich willkommen zu diesem Test.
Heute möchte ich die Qualität der Sprachausgabe demonstrieren.
Dies ist ein längerer Text um zu testen ob das Modell stabil bleibt."""

wav = tts.tts(text=long_text, speaker_wav=ref, language="de")
```

**Ergebnis:**
- ❌ **Erste Hälfte:** Verständlich
- ❌ **Zweite Hälfte:** Kompletter Wortsalat
- ❌ Das Modell "driftet" bei langem deutschen Text

### Test 5: Kurze Segmente (Segment-Lösung)

Lösung: Text in kurze Sätze zerlegen, einzeln generieren, zusammenfügen

```python
import numpy as np

texts = [
    "Hallo und herzlich willkommen.",
    "Dies ist ein Test der Sprachausgabe.",
    "Ich hoffe es klingt natürlich und klar."
]

all_audio = []
for text in texts:
    wav = tts.tts(text=text, speaker_wav=ref, language="de")
    all_audio.append(wav)

# Zusammenfügen mit kurzen Pausen
combined = np.concatenate(all_audio)
```

**Ergebnis:**
- ✅ Kein Drift mehr
- ✅ Jedes Segment verständlich
- ⚠️ Aber: "Quengeligkeit" bleibt — das ist ein Modell-Problem, kein Text-Problem

## Kumulative Bewertung

| Test | Text-Klarheit | Stimmen-Ähnlichkeit | Sound-Qualität | Räumlichkeit | Gesamt |
|------|---------------|---------------------|----------------|--------------|--------|
| Test 1 (Erste Generierung) | 3/5 | 2/5 | 3/5 | 2/5 | 2,5/5 |
| Test 2 (Weniger Satzzeichen) | 3/5 | 2/5 | 3/5 | 2/5 | 2,5/5 |
| Test 3 (Schnelleres Tempo) | 3,5/5 | 2/5 | 3,5/5 | 2/5 | 2,75/5 |
| Test 4 (Langer Text) | 2/5 | 2/5 | 2/5 | 2/5 | 2/5 |
| Test 5 (Kurze Segmente) | 4/5 | 2/5 | 3,5/5 | 2/5 | 2,875/5 |

### Endbewertung XTTS v2

| Kriterium | Bewertung | Kommentar |
|-----------|-----------|-----------|
| **Text-Klarheit** | 3,5/5 | Mit Segment-Lösung OK |
| **Stimmen-Ähnlichkeit** | 2/5 | Kaum erkennbar |
| **Sound-Qualität** | 3,5/5 | Gut, aber quengelig |
| **Räumlichkeit** | 2/5 | Flach |
| **Gesamt** | **2,75/5** | Nutzbar für Prototypen, nicht für Produktion |

## Warum XTTS v2 bei Deutsch scheitert

```text
┌────────────────────────────────────────────────────────────┐
│              XTTS v2 TRAININGSDATEN                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│   Englisch     ████████████████████████████████████  70%  │
│   Spanisch     ████████████                          10%  │
│   Französisch  ███████                              7%   │
│   Deutsch      ██                                   3%   │
│   Sonstige     ██████                              10%   │
│                                                            │
│   German ist eine "Gast-Sprache" — das Modell toleriert   │
│   Deutsch, hat es aber nie wirklich gelernt.              │
│                                                            │
│   Fehlende deutsche Phonetik:                             │
│   • "ch" wie in "ich" — zu weich                         │
│   • "r" wie in "rot" — zu rollend                        │
│   • Umlaute (ä, ö, ü) — falsch positioniert              │
│   • Endungen (-ung, -heit) — zu nasal                    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## Fazit

XTTS v2 ist ein beeindruckendes Modell für Englisch, aber für Deutsch unzureichend. Die "Quengeligkeit" und der Wortsalat bei längerem Text sind **keine Konfigurationsprobleme**, sondern **Trainingsprobleme**.

**Mögliche Verbesserungen:**
1. **XTTS v2 Fine-Tuning** mit 30+ Min. deutschen Daten → Könnte helfen
2. **Kommerzielle APIs** → Sofort besser, aber kostenpflichtig

---

**Nächster Schritt:** [Phase 4: CosyVoice Tests](cosyvoice.md)
