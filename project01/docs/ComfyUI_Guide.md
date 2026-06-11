# ComfyUI Installation and Usage Guide

This is a guide to install and run ComfyUI with Flux 2.dev locally.

---

## Installation

### ComfyUI Installation

```bash
# Clone the repository and install dependencies
cd /your/path/for/installation
git clone https://github.com/Comfy-Org/ComfyUI.git
cd ComfyUI

# Create venv, activate it, and install requirements
python3 -m venv venv
source ./venv/bin/activate

# Install packages
pip install -r requirements.txt
pip install --upgrade accelerate diffusers
pip install --upgrade huggingface_hub
```

---

## Installing Models

### Installing Models for Flux 1.dev (fp8)

```bash
# Move to the ComfyUI folder
cd /your/path/ComfyUI

# Flux model
wget -O models/diffusion_models/flux1-dev-fp8.safetensors https://huggingface.co/Comfy-Org/flux1-dev/resolve/main/flux1-dev-fp8.safetensors

# Text encoder
wget -O models/text_encoders/flux1-dev-clip_l.safetensors https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/clip_l.safetensors
wget -O models/text_encoders/flux1-dev-t5xxl_fp8_e4m3fn_scaled.safetensors https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/t5xxl_fp8_e4m3fn_scaled.safetensors

# Variational auto encoder
wget -O models/vae/flux1-dev-ae.safetensors https://huggingface.co/Comfy-Org/Lumina_Image_2.0_Repackaged/resolve/main/split_files/vae/ae.safetensors
```

### Installing Models for Flux 2.dev (fp8)

```bash
# Move to the ComfyUI folder
cd /your/path/ComfyUI

# Flux model
wget -O models/diffusion_models/flux2_dev_fp8mixed.safetensors https://huggingface.co/Comfy-Org/flux2-dev/resolve/main/split_files/diffusion_models/flux2_dev_fp8mixed.safetensors

# Text encoder
wget -O models/text_encoders/flux2-dev-mistral_3_small_flux2_fp8.safetensors https://huggingface.co/Comfy-Org/flux2-dev/resolve/main/split_files/text_encoders/mistral_3_small_flux2_fp8.safetensors

# Variational auto encoder
wget -O models/vae/flux2-dev-vae.safetensors https://huggingface.co/Comfy-Org/flux2-dev/resolve/main/split_files/vae/flux2-vae.safetensors
```

### Installing Stable Diffusion XL 1.0

```bash
# Move to the ComfyUI folder
cd /your/path/ComfyUI

# Base model
wget -O models/checkpoints/sd_xl_base_1.0.safetensors https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors

# Refiner model
wget -O models/checkpoints/sd_xl_refiner_1.0.safetensors https://huggingface.co/stabilityai/stable-diffusion-xl-refiner-1.0/resolve/main/sd_xl_refiner_1.0.safetensors
```

### Installing Stable Diffusion 3.5 Large

1. Create a Hugging Face account
2. Go to [this repository](https://huggingface.co/stabilityai/stable-diffusion-3.5-large/tree/main) and request access
3. Generate a User Access Token on the Hugging Face website and save it

```bash
# Move to the ComfyUI folder
cd /your/path/ComfyUI

# Activate environment
source ./venv/bin/activate

# Login to your account by pasting in your Access Token
hf auth login

# Base model
hf download stabilityai/stable-diffusion-3.5-large sd3.5_large.safetensors --local-dir models/checkpoints/

# Text encoders
wget -O models/text_encoders/sd3_5_clip_l.safetensors https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8/resolve/main/text_encoders/clip_l.safetensors
wget -O models/text_encoders/sd3_5_clip_g.safetensors https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8/resolve/main/text_encoders/clip_g.safetensors
wget -O models/text_encoders/sd3_5_t5xxl_fp8_e4m3fn_scaled.safetensors https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8/resolve/main/text_encoders/t5xxl_fp8_e4m3fn_scaled.safetensors
```

---

## Using ComfyUI

### Start ComfyUI

```bash
# Move to the ComfyUI directory
cd /your/path/ComfyUI

# If not enabled, enable the venv
source ./venv/bin/activate

# Start ComfyUI
python3 main.py
```

### Opening ComfyUI

#### Desktop
When using on a desktop PC, open this URL in your browser:
```bash
http://127.0.0.1:8188
```

#### VM/Server
When using a VM/Server, you need a second terminal to forward the port to your machine:

```bash
# Connect to your VM and keep the terminal open
ssh -L 8188:127.0.0.1:8188 username@hostname_or_ip

# Open this URL in your desktop's browser
http://127.0.0.1:8188
```

---

## Generate Images with ComfyUI

### Node Setup: Flux 1.dev (fp8)

```bash
# Copy the template into the workflow folder
cp /your/path/to/repo/Workflows/ImageGeneration/Flux1DevImageGeneration.json /your/path/ComfyUI/user/default/workflows/
```

In ComfyUI, open the workflow in the sidebar → Workflows. To generate an image, edit the prompt in the green node `CLIP Text Encode (positive prompt)` and press the **Run** button at the top right.

### Node Setup: Flux 2.dev (fp8)

```bash
# Copy the template into the workflow folder
cp /your/path/to/repo/Workflows/ImageGeneration/Flux2DevImageGeneration.json /your/path/ComfyUI/user/default/workflows/
```

In ComfyUI, open the workflow in the sidebar → Workflows. To generate an image, edit the prompt in the green node `CLIP Text Encode (positive prompt)` and press the **Run** button at the top right.

### Node Setup: Stable Diffusion XL 1.0

```bash
# Copy the template into the workflow folder
cp /your/path/to/repo/Workflows/ImageGeneration/SDXL1ImageGeneration.json /your/path/ComfyUI/user/default/workflows/
```

In ComfyUI, open the workflow in the sidebar → Workflows. To generate an image, edit the prompt in the green node `Positive Prompt (Text)` and press the **Run** button at the top right.

### Node Setup: Stable Diffusion 3.5 Large

```bash
# Copy the template into the workflow folder
cp /your/path/to/repo/Workflows/ImageGeneration/SD35LargeImageGeneration.json /your/path/ComfyUI/user/default/workflows/
```

In ComfyUI, open the workflow in the sidebar → Workflows. To generate an image, edit the prompt in the green node `Positive Prompt` and press the **Run** button at the top right.

---

## Auto-Caption Images

### Installation

```bash
# Move to directory
cd /your/path/ComfyUI

# Install packages
cd custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack.git
git clone https://github.com/yolain/ComfyUI-Easy-Use.git
git clone https://github.com/kijai/ComfyUI-Florence2.git
git clone https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite.git
git clone https://github.com/ltdrdata/was-node-suite-comfyui.git
git clone https://github.com/M1kep/Comfy_KepListStuff.git

cd ..

# Activate environment
source ./venv/bin/activate

# Download the Florence-2 repository
hf download microsoft/Florence-2-large --local-dir models/LLM/Florence-2-large
```

### Run Captioning

1. Open ComfyUI as normal (restart after installing packages)
2. Copy workflow into workflow folder:
   ```bash
   cp /your/path/to/repo/Workflows/CaptionImages/CaptionImages.json /your/path/ComfyUI/user/default/workflows/
   ```
3. Open the workflow in ComfyUI
4. Configure the following nodes:
   - `Input Folder with images` → Path to the folder with images to caption
   - `Dataset Name` → `NumberOfEpochs_TriggerWord`
   - `Word to Replace` → Description of subject that's being replaced with trigger word (man, woman, etc.)
   - `Trigger Word` → Must be the same as in step 2
5. Run the inference
6. Copy the dataset to your desired location:
   ```bash
   cp -r /your/path/ComfyUI/outputs/datasetname /your/path/

   # Location for later LoRA training
   cp -r /your/path/ComfyUI/outputs/datasetname /your/path/ComfyUI/inputs/
   ```

---

## Training a LoRA for Stable Diffusion XL 1.0

### Setup

```bash
# Copy dataset into input folder (the number_triggerword folder) if not already done in auto-captioning
cp -r /your/path/to/dataset /your/path/ComfyUI/inputs/
```

### Training LoRA

```bash
# Copy the template into the workflow folder
cp /your/path/to/repo/Workflows/LoRATraining/SDXL1TrainLoRA.json /your/path/ComfyUI/user/default/workflows/
```

1. Run the workflow
2. After training, move the LoRA into the LoRA folder:
   ```bash
   mv /your/path/ComfyUI/outputs/loras/yourLoRAName.safetensors /your/path/ComfyUI/models/loras/yourLoRAName.safetensors
   ```

---

## Training a LoRA for Flux 1.dev (fp8)

### Installing Correct Model for LoRA Training

```bash
# Move to the ComfyUI folder
cd /your/path/ComfyUI

# Activate environment
source ./venv/bin/activate

# Login to your account by pasting in your Access Token from your Hugging Face account
# Also request access with your account for this repo: https://huggingface.co/black-forest-labs/FLUX.1-dev/tree/main
hf auth login

hf download black-forest-labs/FLUX.1-dev flux1-dev.safetensors --local-dir models/diffusion_models/
```

To train a LoRA for Flux 1.dev, see the [**Kohya SS Guide**](Kohya_SS.md).

---

## Generating Images with LoRA

### Stable Diffusion XL 1.0

1. Make sure to have a copy of the LoRA in the `/your/path/ComfyUI/models/loras/` folder. If you did not train your own LoRA, use the one in this repository.

```bash
# Only when using LoRA from this repo
cp /your/path/to/repo/LoRA_Models/UweHahneSDXL_v1.safetensors /your/path/ComfyUI/models/loras/

# Copy workflow
cp /your/path/to/repo/Workflows/LoRAImageGeneration/SDXL1LoRAImageGeneration.json /your/path/ComfyUI/user/default/workflows/

# Open workflow and generate images as usual
```

### Flux 1.dev

1. Make sure you have a copy of the trained LoRA in this folder `/your/path/ComfyUI/models/loras/`. You can also use the LoRA from this repo.

```bash
# Only when using LoRA from this repo
cp /your/path/to/repo/LoRA_Models/UweHahneFlux1_v1.safetensors /your/path/ComfyUI/models/loras/

# Copy workflow
cp /your/path/to/repo/Workflows/LoRAImageGeneration/Flux1LoRAImageGeneration.json /your/path/ComfyUI/user/default/workflows/

# Open workflow and generate images as usual
```
