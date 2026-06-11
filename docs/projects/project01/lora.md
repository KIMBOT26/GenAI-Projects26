# LoRA Training Guide

Train custom LoRA adapters for personalized image generation.

---

## Overview

LoRA (Low-Rank Adaptation) allows you to fine-tune diffusion models with a small dataset. This guide covers training for:

- **SDXL 1.0** — Using ComfyUI workflows
- **Flux 1.dev** — Using Kohya sd-scripts

---

## Prerequisites

- **Dataset:** 10-20 images with captions (`.txt` files)
- **GPU:** Minimum 8GB VRAM (12GB+ recommended)
- **Disk space:** ~20GB for training environment

---

## Dataset Preparation

### Option 1: Auto-Captioning with Florence-2

1. Install required custom nodes:
   ```bash
   cd /your/path/ComfyUI/custom_nodes
   git clone https://github.com/kijai/ComfyUI-Florence2.git
   # ... other nodes from ComfyUI_Guide.md
   ```

2. Download Florence-2 model:
   ```bash
   hf download microsoft/Florence-2-large --local-dir models/LLM/Florence-2-large
   ```

3. Run the captioning workflow:
   ```bash
   cp /your/path/to/repo/Workflows/CaptionImages/CaptionImages.json \
      /your/path/ComfyUI/user/default/workflows/
   ```

4. Configure nodes:
   - `Input Folder with images` → Your images folder
   - `Dataset Name` → `10_TriggerWord` (epochs_triggerword)
   - `Word to Replace` → Subject description (e.g., "man", "woman")
   - `Trigger Word` → Your chosen trigger word

### Option 2: Manual Captions

Create `.txt` files alongside each image:
```
image_00001.png
image_00001.txt  ← Contains: "A photo of TriggerWord person, smiling"
```

---

## Training SDXL 1.0 LoRA

### Using ComfyUI Workflow

1. Copy dataset to ComfyUI inputs:
   ```bash
   cp -r /your/path/dataset /your/path/ComfyUI/inputs/
   ```

2. Load the training workflow:
   ```bash
   cp /your/path/to/repo/Workflows/LoRATraining/SDXL1TrainLoRA.json \
      /your/path/ComfyUI/user/default/workflows/
   ```

3. Run the workflow in ComfyUI

4. Move trained LoRA:
   ```bash
   mv /your/path/ComfyUI/outputs/loras/yourLoRAName.safetensors \
      /your/path/ComfyUI/models/loras/
   ```

---

## Training Flux 1.dev LoRA

### Setup Kohya sd-scripts

```bash
# Install Conda and CUDA 12.6 first
cd /your/path
git clone --recursive https://github.com/kohya-ss/sd-scripts.git
cd sd-scripts

# Create environment
conda create --name venv python=3.10
conda activate venv
pip install -r "requirements.txt"

# Install PyTorch for CUDA 12.6
pip install torch==2.11.0 torchvision==0.26.0 torchaudio==2.11.0 \
  --index-url https://download.pytorch.org/whl/cu126
```

### Configure GPU

```bash
cd /your/path/sd-scripts
conda activate venv
accelerate config  # Select single GPU, no optimization
```

### Prepare Dataset

```bash
cd /your/path/sd-scripts
mkdir dataset output DatasetConfig

# Copy dataset
cp -r /your/path/ComfyUI/inputs/datasetname dataset/

# Copy config
cp /your/path/to/repo/DatasetConfig/DatasetConfig.toml DatasetConfig/

# Edit config
nano DatasetConfig/DatasetConfig.toml
```

### Install Flux Models

```bash
cd /your/path/ComfyUI
source ./venv/bin/activate
hf auth login

hf download black-forest-labs/FLUX.1-dev flux1-dev.safetensors \
  --local-dir models/diffusion_models/
```

### Run Training

```bash
cd /your/path/sd-scripts
conda activate venv

accelerate launch --num_processes=1 --num_cpu_threads_per_process 1 \
  flux_train_network.py \
  --pretrained_model_name_or_path="/your/path/ComfyUI/models/diffusion_models/flux1-dev.safetensors" \
  --clip_l="/your/path/ComfyUI/models/text_encoders/flux1-dev-clip_l.safetensors" \
  --t5xxl="/your/path/ComfyUI/models/text_encoders/flux1-dev-t5xxl_fp8_e4m3fn_scaled.safetensors" \
  --ae="/your/path/ComfyUI/models/vae/flux1-dev-ae.safetensors" \
  --dataset_config="/your/path/sd-scripts/DatasetConfig/DatasetConfig.toml" \
  --output_dir="/your/path/sd-scripts/output" \
  --output_name="YourLoRAName" \
  --save_model_as=safetensors \
  --network_module=networks.lora_flux \
  --network_dim=64 \
  --network_alpha=64 \
  --learning_rate=3e-4 \
  --optimizer_type="AdamW8bit" \
  --lr_scheduler="constant" \
  --sdpa \
  --max_train_epochs=25 \
  --save_every_n_epochs=1 \
  --mixed_precision="bf16" \
  --gradient_checkpointing \
  --guidance_scale=1.0 \
  --timestep_sampling="flux_shift" \
  --model_prediction_type="raw" \
  --blocks_to_swap=18 \
  --cache_text_encoder_outputs \
  --cache_latents \
  --fp8_base
```

### Copy Trained Model

```bash
cp /your/path/sd-scripts/output/yourLoRAName.safetensors \
   /your/path/ComfyUI/models/loras/
```

---

## Using Your LoRA

### In ComfyUI

1. Ensure LoRA is in `models/loras/`
2. Load a LoRA inference workflow:
   ```bash
   # SDXL
   cp /your/path/to/repo/Workflows/LoRAImageGeneration/SDXL1LoRAImageGeneration.json \
      /your/path/ComfyUI/user/default/workflows/

   # Flux
   cp /your/path/to/repo/Workflows/LoRAImageGeneration/Flux1LoRAImageGeneration.json \
      /your/path/ComfyUI/user/default/workflows/
   ```
3. Select your LoRA in the workflow
4. Use your trigger word in the prompt

### Example Prompt

```
A photo of TriggerWord person, standing in a modern office, professional lighting
```

---

## Training Tips

| Parameter | Recommendation |
|-----------|----------------|
| **Dataset size** | 10-20 images minimum |
| **Epochs** | 10-25 (more = overfitting risk) |
| **Learning rate** | 3e-4 (AdamW8bit) |
| **Network dim** | 64 (higher = more detail) |
| **Batch size** | 1-2 (depends on VRAM) |

---

## Troubleshooting

**CUDA Out of Memory:**
- Reduce batch size to 1
- Enable `--gradient_checkpointing`
- Use `--mixed_precision="bf16"`

**LoRA Not Loading:**
- Verify file is in `models/loras/`
- Restart ComfyUI
- Check workflow has LoRA node connected

**Poor Results:**
- Increase dataset size
- Adjust trigger word (use unique word)
- Reduce training epochs

---

## Next Steps

- **[Sample Models →](../../project01/LoRA_Models/)** — Pre-trained LoRAs included
- **[Full Kohya Guide →](../../project01/docs/Kohya_SS.md)** — Detailed commands
- **[Results →](../../project01/Results/)** — Example outputs
