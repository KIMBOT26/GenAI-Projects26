# Workflows Guide

ComfyUI workflows for image generation with different models.

---

## Overview

This project includes pre-configured workflows for all supported models. Each workflow is a JSON file that can be imported directly into ComfyUI.

| Workflow | Model | File |
|----------|-------|------|
| Flux 1.dev | Flux 1.dev fp8 | `Flux1DevImageGeneration.json` |
| Flux 2.dev | Flux 2.dev fp8 | `Flux2DevImageGeneration.json` |
| SDXL 1.0 | SDXL Base + Refiner | `SDXL1ImageGeneration.json` |
| SD 3.5 Large | SD 3.5 Large | `SD35LargeImageGeneration.json` |

---

## Loading Workflows

1. Copy the workflow to your ComfyUI workflows folder:
   ```bash
   cp /your/path/to/repo/Workflows/ImageGeneration/Flux1DevImageGeneration.json \
      /your/path/ComfyUI/user/default/workflows/
   ```

2. In ComfyUI, open the **sidebar** → **Workflows** → Select your workflow

3. Click **Run** (top right) to generate

---

## Flux 1.dev Workflow

### Setup
```bash
cp /your/path/to/repo/Workflows/ImageGeneration/Flux1DevImageGeneration.json \
   /your/path/ComfyUI/user/default/workflows/
```

### Usage
1. Open the workflow in ComfyUI
2. Edit the prompt in the green node `CLIP Text Encode (positive prompt)`
3. Press **Run**

### Required Models
- `flux1-dev-fp8.safetensors`
- `flux1-dev-clip_l.safetensors`
- `flux1-dev-t5xxl_fp8_e4m3fn_scaled.safetensors`
- `flux1-dev-ae.safetensors`

---

## Flux 2.dev Workflow

### Setup
```bash
cp /your/path/to/repo/Workflows/ImageGeneration/Flux2DevImageGeneration.json \
   /your/path/ComfyUI/user/default/workflows/
```

### Usage
1. Open the workflow in ComfyUI
2. Edit the prompt in `CLIP Text Encode (positive prompt)`
3. Press **Run**

### Required Models
- `flux2_dev_fp8mixed.safetensors`
- `flux2-dev-mistral_3_small_flux2_fp8.safetensors`
- `flux2-dev-vae.safetensors`

---

## SDXL 1.0 Workflow

### Setup
```bash
cp /your/path/to/repo/Workflows/ImageGeneration/SDXL1ImageGeneration.json \
   /your/path/ComfyUI/user/default/workflows/
```

### Usage
1. Open the workflow in ComfyUI
2. Edit the prompt in `Positive Prompt (Text)`
3. Press **Run**

### Required Models
- `sd_xl_base_1.0.safetensors`
- `sd_xl_refiner_1.0.safetensors`

---

## SD 3.5 Large Workflow

### Setup
```bash
cp /your/path/to/repo/Workflows/ImageGeneration/SD35LargeImageGeneration.json \
   /your/path/ComfyUI/user/default/workflows/
```

### Usage
1. Open the workflow in ComfyUI
2. Edit the prompt in `Positive Prompt`
3. Press **Run**

### Required Models
- `sd3.5_large.safetensors`
- `sd3_5_clip_l.safetensors`
- `sd3_5_clip_g.safetensors`
- `sd3_5_t5xxl_fp8_e4m3fn_scaled.safetensors`

---

## Additional Workflows

### Auto-Captioning
For automatic image captioning with Florence-2:
```bash
cp /your/path/to/repo/Workflows/CaptionImages/CaptionImages.json \
   /your/path/ComfyUI/user/default/workflows/
```

### LoRA Training (SDXL)
```bash
cp /your/path/to/repo/Workflows/LoRATraining/SDXL1TrainLoRA.json \
   /your/path/ComfyUI/user/default/workflows/
```

### LoRA Inference
```bash
# SDXL
cp /your/path/to/repo/Workflows/LoRAImageGeneration/SDXL1LoRAImageGeneration.json \
   /your/path/ComfyUI/user/default/workflows/

# Flux
cp /your/path/to/repo/Workflows/LoRAImageGeneration/Flux1LoRAImageGeneration.json \
   /your/path/ComfyUI/user/default/workflows/
```

---

## Tips

- **Save your modifications:** After editing a workflow, save it with a new name
- **Batch generation:** Modify the batch count in the sampler node
- **Resolution:** Adjust in the Empty Latent Image node
- **Seed:** Set a fixed seed for reproducible results

---

## Next Steps

- **[LoRA Training →](lora.md)** — Train custom models
- **[Full Workflows →](../../project01/Workflows/)** — Browse all workflow files
