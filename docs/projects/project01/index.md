# Projekt 1 — ComfyUI: Text-to-Image & LoRA Training

> **Status:** ✅ Complete — Full pipeline from installation to LoRA training  
> **Focus:** Stable Diffusion XL 1.0, Flux 1.dev, Flux 2.dev, SD 3.5 Large  
> **Institution:** Hochschule Furtwangen (HFU), Faculty I: Computer Science & Applications  
> **Project Lead:** Prof. Dr. Uwe Hahne

---

## What is This?

A comprehensive guide for setting up **ComfyUI** with multiple diffusion models (Flux, SDXL, SD 3.5) and training custom **LoRA adapters** for personalized image generation. Includes auto-captioning workflows, dataset preparation, and production-ready inference pipelines.

---

## Quick Navigation

| Section | Description |
|---------|-------------|
| [**Installation**](installation.md) | ComfyUI setup and model installation |
| [**Workflows**](workflows.md) | Image generation workflows for all models |
| [**LoRA Training**](lora.md) | Train custom LoRAs for SDXL and Flux |

---

## Repository Structure

```
project01/
├── README.md                    ← Main documentation
├── docs/
│   ├── ComfyUI_Guide.md         ← Detailed installation guide
│   └── Kohya_SS.md              ← LoRA training with Kohya sd-scripts
├── DatasetConfig/               ← LoRA training configuration
├── LoRA_Dataset/                ← Sample datasets
├── LoRA_Models/                 ← Pre-trained LoRA models
├── Results/                     ← Sample generated images
├── TrainingImages/              ← Raw training images
└── Workflows/                   ← ComfyUI workflow JSONs
```

---

## Supported Models

| Model | Type | Precision | Use Case |
|-------|------|-----------|----------|
| **Flux 1.dev** | Diffusion | fp8 | High-quality text-to-image |
| **Flux 2.dev** | Diffusion | fp8 mixed | Latest Flux with Mistral encoder |
| **SDXL 1.0** | Diffusion | fp16 | Fast generation, LoRA training |
| **SD 3.5 Large** | Diffusion | fp8 | Best quality, requires HF access |

---

## Key Features

- ✅ **Multi-model support** — Flux, SDXL, SD 3.5 in one setup
- ✅ **LoRA training** — Custom adapters for personalized generation
- ✅ **Auto-captioning** — Florence-2 based dataset preparation
- ✅ **Production workflows** — Ready-to-use JSON workflows
- ✅ **Sample outputs** — Included results for reference

---

## Quick Start

```bash
# Clone the repository
cd /your/path
git clone https://github.com/yourusername/GenAI-Projects26.git
cd GenAI-Projects26/project01

# Follow the installation guide
# See: Installation →
```

---

## Documentation Links

- **[Installation Guide →](installation.md)** — Full ComfyUI setup
- **[Workflows →](workflows.md)** — Model-specific workflows
- **[LoRA Training →](lora.md)** — Train custom models
- **[Main README →](../../project01/README.md)** — Complete documentation

---

*Prof. Dr. Uwe Hahne @ Hochschule Furtwangen | ComfyUI & LoRA Training Project*
