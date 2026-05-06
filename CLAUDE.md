# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Environment Setup

```bash
# CPU environment (Python 3.14)
./venv/Scripts/activate

# GPU environment (Python 3.13 + PyTorch Nightly - REQUIRED for RTX 50-series)
./venv_gpu/Scripts/activate

# Install new dependencies in GPU environment
./venv_gpu/Scripts/pip install peft accelerate opacus
```

### Running Privacy Audits

```bash
# Membership Inference Attack (MIA)
python run_mia.py --cf configs/cifar10.yaml

# Range Membership Inference Attack (RaMIA)
python run_range_mia.py --cf configs/ramia/cifar10.yaml

# Dataset Usage Cardinality Inference (DUCI)
python run_duci.py --cf configs/duci/cifar10.yaml

# Differential Privacy Audit
python run_audit_dp.py --cf configs/dpaudit/cifar10_dp_train_natural_1000.yaml
```

### Training New Models vs. Reusing Existing

The `log_dir` in config determines model behavior:
- If `log_dir/models/models_metadata.json` exists → loads existing models
- If not found → trains new models and saves to `log_dir/models/`

To force retraining: delete `models/` subdirectory or change `log_dir`

```bash
# Delete old models to force retraining
rm -rf demo_cifar10/models/
```

### Jupyter Notebooks

```bash
jupyter notebook
# Open: demo.ipynb, demo_ramia.ipynb, demo_audit_dp.ipynb, or demo_duci.ipynb
```

## Architecture Overview

```
Entry Points (run_*.py)
    ↓
Main Pipeline (audit.py, util.py, get_signals.py)
    ↓
┌───────────┬────────────┬─────────────────┬────────────────┐
│           │            │                 │                │
Models/    Trainers/   Dataset/         Modules/        Attacks.py
utils.py   *.py        utils.py         mia/            rmia.py
           │            │                 ramia/          loss.py
           │            │                 duci/
           │            │                 └─ range_samplers/
```

### Key Components

**Entry Points:**
- `run_mia.py` - Membership Inference Attack (privacy leakage through training points)
- `run_range_mia.py` - Range MIA (leakage near training points)
- `run_duci.py` - Dataset Usage Cardinality Inference (percentage of dataset used)
- `run_audit_dp.py` - DP lower bounds auditing

**Core Logic:**
- `audit.py` - Main audit orchestration, ROC computation, result aggregation
- `get_signals.py` - Signal extraction from models (softmax outputs, embeddings)
- `util.py` - Dataset loading, config validation, directory management
- `attacks.py` - Attack implementations (RMIA, LOSS) with offline parameter tuning

**Model Handling:**
- `models/utils.py` - `get_model()`, `load_models()`, `train_models()`
- `models/*.py` - CNN, WideResNet, MLP, AlexNet architectures
- `trainers/default_trainer.py` - Standard PyTorch training loop
- `trainers/train_transformers.py` - HuggingFace transformers training with PEFT/LoRA

**Modules:**
- `modules/mia/` - Standard membership inference attacks (RMIA, LOSS)
- `modules/ramia/` - Range membership inference with geometric/L2/word-replacement samplers
- `modules/duci/` - Dataset usage cardinality inference

**Dataset Handling:**
- `dataset/utils.py` - `get_dataset()` factory, dataloader creation
- `dataset/range_dataset.py` - RangeDataset wrapper for RaMIA
- `dataset/tabular.py`, `agnews.py` - Tabular and text dataset loaders

### Configuration System (YAML)

All runs are driven by YAML configs in `configs/`:

**Critical config fields:**
- `run.log_dir` - Determines model reuse vs. training
- `train.device` - "cuda:0" or "cpu" (training)
- `audit.device` - "cuda:0" or "cpu" (inference)
- `train.epochs` - Number of training epochs
- `audit.num_ref_models` - Reference models for attack comparison
- `audit.data_size` - Audit dataset size (must be even)

**Attack selection:**
- `audit.algorithm` - "RMIA" (recommended) or "LOSS"

## RTX 50-Series GPU Support

**Critical:** RTX 5060 Ti (sm_120 / Blackwell) is NOT supported in stable PyTorch (2.6.x).

**Required solution:** Use PyTorch Nightly with CUDA 12.8
```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128
```

Without Nightly: `RuntimeError: CUDA error: no kernel image is available for execution on the device`

See `GPU_SETUP.md` for complete setup instructions.

## Output Structure

```
<log_dir>/
├── models/
│   ├── models_metadata.json    # Model metadata and paths
│   ├── model_*.pkl             # Trained models
│   └── memberships.npy         # Training membership labels
├── report/
│   ├── exp/                    # Per-model ROC curves
│   ├── attack_result_average.csv
│   ├── ROC_(log_)average.png
│   └── log_time_analysis.log
└── signals/                    # Cached attack signals
```

## Extending the Tool

**New datasets:**
1. Create `dataset/<dataset_name>.py`
2. Add to `dataset/utils.py:get_dataset()`
3. Update `INPUT_OUTPUT_SHAPE` in `models/utils.py` if needed

**New models:**
1. Create architecture in `models/<model>.py`
2. Add to `models/utils.py:get_model()`
3. Optionally add custom trainer in `trainers/`

**New range samplers (for RaMIA):**
1. Create `modules/ramia/range_samplers/sample_<name>.py`
2. Implement `sample()` and `_sample()` methods
3. Update config `ramia.range_function`
