# GPU Setup for ML Privacy Meter

## Environment with GPU Support

### Using the GPU-enabled Virtual Environment

```bash
# Activate the GPU environment (Python 3.13 + PyTorch CUDA 12.4)
./venv_gpu/Scripts/activate

# Run with GPU
python run_mia.py --cf configs/cifar10.yaml
```

### GPU Info
- **GPU:** NVIDIA GeForce RTX 5060 Ti (16GB VRAM)
- **CUDA Version:** 12.4
- **PyTorch:** 2.6.0+cu124

### Note on RTX 50-Series
RTX 5060 Ti is very new (sm_120 architecture). You may see a warning about compatibility, but GPU operations work correctly.

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
