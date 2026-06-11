# Installation Guide

Complete setup instructions for ComfyUI and all supported models.

---

## Prerequisites

- **Python 3.10+**
- **Git**
- **CUDA 12.6** (for GPU acceleration)
- **~50 GB free disk space** (for models)

---

## ComfyUI Installation

```bash
# Clone the repository and install dependencies
cd /your/path/for/installation
git clone https://github.com/Comfy-Org/ComfyUI.git
cd ComfyUI

# Create virtual environment
python3 -m venv venv
source ./venv/bin/activate

# Install requirements
pip install -r requirements.txt
pip install --upgrade accelerate diffusers huggingface_hub
```

---

## Model Installation

### Flux 1.dev (fp8)

```bash
cd /your/path/ComfyUI

# Flux model
wget -O models/diffusion_models/flux1-dev-fp8.safetensors \
  https://huggingface.co/Comfy-Org/flux1-dev/resolve/main/flux1-dev-fp8.safetensors

# Text encoders
wget -O models/text_encoders/flux1-dev-clip_l.safetensors \
  https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/clip_l.safetensors
wget -O models/text_encoders/flux1-dev-t5xxl_fp8_e4m3fn_scaled.safetensors \
  https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/t5xxl_fp8_e4m3fn_scaled.safetensors

# VAE
wget -O models/vae/flux1-dev-ae.safetensors \
  https://huggingface.co/Comfy-Org/Lumina_Image_2.0_Repackaged/resolve/main/split_files/vae/ae.safetensors
```

### Flux 2.dev (fp8)

```bash
cd /your/path/ComfyUI

# Flux model
wget -O models/diffusion_models/flux2_dev_fp8mixed.safetensors \
  https://huggingface.co/Comfy-Org/flux2-dev/resolve/main/split_files/diffusion_models/flux2_dev_fp8mixed.safetensors

# Text encoder (Mistral)
wget -O models/text_encoders/flux2-dev-mistral_3_small_flux2_fp8.safetensors \
  https://huggingface.co/Comfy-Org/flux2-dev/resolve/main/split_files/text_encoders/mistral_3_small_flux2_fp8.safetensors

# VAE
wget -O models/vae/flux2-dev-vae.safetensors \
  https://huggingface.co/Comfy-Org/flux2-dev/resolve/main/split_files/vae/flux2-vae.safetensors
```

### SDXL 1.0

```bash
cd /your/path/ComfyUI

# Base model
wget -O models/checkpoints/sd_xl_base_1.0.safetensors \
  https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors

# Refiner model
wget -O models/checkpoints/sd_xl_refiner_1.0.safetensors \
  https://huggingface.co/stabilityai/stable-diffusion-xl-refiner-1.0/resolve/main/sd_xl_refiner_1.0.safetensors
```

### SD 3.5 Large

!!! warning "Hugging Face Access Required"
    You need to request access on Hugging Face before downloading.

1. Create a Hugging Face account
2. Request access: [stabilityai/stable-diffusion-3.5-large](https://huggingface.co/stabilityai/stable-diffusion-3.5-large/tree/main)
3. Generate an Access Token

```bash
cd /your/path/ComfyUI
source ./venv/bin/activate

# Login
hf auth login

# Download model
hf download stabilityai/stable-diffusion-3.5-large sd3.5_large.safetensors \
  --local-dir models/checkpoints/

# Text encoders
wget -O models/text_encoders/sd3_5_clip_l.safetensors \
  https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8/resolve/main/text_encoders/clip_l.safetensors
wget -O models/text_encoders/sd3_5_clip_g.safetensors \
  https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8/resolve/main/text_encoders/clip_g.safetensors
wget -O models/text_encoders/sd3_5_t5xxl_fp8_e4m3fn_scaled.safetensors \
  https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8/resolve/main/text_encoders/t5xxl_fp8_e4m3fn_scaled.safetensors
```

---

## Starting ComfyUI

```bash
cd /your/path/ComfyUI
source ./venv/bin/activate
python3 main.py
```

### Accessing the UI

**Desktop:** Open `http://127.0.0.1:8188` in your browser

**VM/Server:** Forward the port via SSH:
```bash
ssh -L 8188:127.0.0.1:8188 username@hostname_or_ip
```

---

## Next Steps

- **[Workflows →](workflows.md)** — Set up image generation workflows
- **[LoRA Training →](lora.md)** — Train custom models
- **[Full Guide →](../../project01/docs/ComfyUI_Guide.md)** — Detailed documentation
