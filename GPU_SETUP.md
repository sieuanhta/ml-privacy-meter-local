# GPU Setup for ML Privacy Meter

## Environment with GPU Support

### Using the GPU-enabled Virtual Environment

```bash
# Activate the GPU environment (Python 3.13 + PyTorch Nightly CUDA 12.8)
./venv_gpu/Scripts/activate

# Run with GPU
python run_mia.py --cf configs/cifar10.yaml
```

### GPU Info
- **GPU:** NVIDIA GeForce RTX 5060 Ti (16GB VRAM)
- **CUDA Capability:** sm_120 (Blackwell Architecture)
- **PyTorch:** 2.12.0.dev20260408+cu128 (Nightly Build)

### IMPORTANT: RTX 50-Series Support
RTX 5060 Ti uses the new Blackwell architecture (sm_120) which is **NOT supported** in stable PyTorch builds (2.6.x).

**Solution:** Use PyTorch Nightly with CUDA 12.8:
```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128
```

Without PyTorch Nightly, you will get:
```
RuntimeError: CUDA error: no kernel image is available for execution on the device
```

## CPU Environment (Fallback)

```bash
# Activate the CPU environment (Python 3.14)
./venv/Scripts/activate

# Config files are set to use CPU if needed
```

## Running with GPU

All config files have been updated to use `cuda:0` by default:
- `configs/cifar10.yaml`
- `configs/ramia/cifar10.yaml`

## Installation Commands (GPU Environment)

```bash
# Create venv with Python 3.13
py -3.13 -m venv venv_gpu

# Activate
./venv_gpu/Scripts/activate

# Install PyTorch Nightly (REQUIRED for RTX 50-series)
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128

# Install dependencies
pip install datasets transformers PyYAML matplotlib tqdm scikit-learn scipy ipywidgets jupyter peft accelerate opacus
```
