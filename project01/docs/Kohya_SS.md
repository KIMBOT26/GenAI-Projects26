# Kohya SS — LoRA Training Guide

---

## Installation

This script requires an installation of `Conda` and `CUDA 12.6`.

```bash
# Move to a folder where training environment should be installed
cd /your/path

# Clone the industry-standard training repository
git clone --recursive https://github.com/kohya-ss/sd-scripts.git
cd sd-scripts

# Create a virtual environment with Python 3.10
conda create --name venv python=3.10
conda activate venv
pip install -r "requirements.txt"

# Install correct PyTorch version for CUDA 12.6
pip install torch==2.11.0 torchvision==0.26.0 torchaudio==2.11.0 --index-url https://download.pytorch.org/whl/cu126
```

---

## Configuration

```bash
cd /your/path/sd-scripts
conda activate venv

# Configure GPU setup (select single GPU setup and no optimization)
accelerate config
```

---

## Prepare Dataset

```bash
cd /your/path/sd-scripts
mkdir dataset
mkdir output
mkdir DatasetConfig

# Copy dataset from ComfyUI (when auto-captioning was done and the output was copied into the inputs folder of ComfyUI)
cp -r /your/path/ComfyUI/inputs/datasetname(NumberOfEpochs_TriggerWord) dataset/

# Copy dataset from this repo if no own dataset was created
cp -r /your/path/to/repo/LoRA_Dataset/10_UweHahne dataset/

# Copy the dataset config file from this repo
cp /your/path/to/repo/DatasetConfig/DatasetConfig.toml DatasetConfig/

# Open the config and change paths and trigger word to match yours
nano DatasetConfig/DatasetConfig.toml
```

---

## Training Different Models

### Flux 1.dev

To run training of this model, please install all models as described in [ComfyUI_Guide.md](ComfyUI_Guide.md).

```bash
cd /your/path/sd-scripts
conda activate venv

# Start with commands
accelerate launch --num_processes=1 --num_cpu_threads_per_process 1 flux_train_network.py \
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

#### Example

```bash
accelerate launch --num_processes=1 --num_cpu_threads_per_process 1 flux_train_network.py \
  --pretrained_model_name_or_path="/mnt/disc/ComfyUI_Manual/ComfyUI/models/diffusion_models/flux1-dev.safetensors" \
  --clip_l="/mnt/disc/ComfyUI_Manual/ComfyUI/models/text_encoders/flux1-dev-clip_l.safetensors" \
  --t5xxl="/mnt/disc/ComfyUI_Manual/ComfyUI/models/text_encoders/flux1-dev-t5xxl_fp8_e4m3fn_scaled.safetensors" \
  --ae="/mnt/disc/ComfyUI_Manual/ComfyUI/models/vae/flux1-dev-ae.safetensors" \
  --dataset_config="/mnt/disc/ComfyUI_Manual/kohya_ss/sd-scripts/DatasetConfig/DatasetConfig.toml" \
  --output_dir="/mnt/disc/ComfyUI_Manual/kohya_ss/sd-scripts/output" \
  --output_name="UweHahneFlux1_v1" \
  --save_model_as=safetensors \
  --network_module=networks.lora_flux \
  --network_dim=64 \
  --network_alpha=64 \
  --learning_rate=3e-4 \
  --optimizer_type="AdamW8bit" \
  --lr_scheduler="constant" \
  --sdpa \
  --max_train_epochs=10 \
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

#### Copy Your Trained Model into ComfyUI

```bash
cp output/yourLoRAName.safetensors /your/path/ComfyUI/models/loras/yourLoRAName.safetensors
```
