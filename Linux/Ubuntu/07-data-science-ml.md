# 07 — Data Science, ML and Deep Learning (Ubuntu)

**Profiles:** data-ml · Depends on 02 (uv).

---

## Step 1 — GPU and driver

```bash
lspci | grep -Ei 'vga|3d'
nvidia-smi 2>/dev/null || echo "no working NVIDIA driver"
```
- **Native Ubuntu + NVIDIA:** `sudo ubuntu-drivers install` (module 01 Step 9), then reboot. The
  signed prebuilt modules work with Secure Boot. Check that `nvidia-smi` shows the GPU.
  **Remember:** always update with `apt full-upgrade`, so a kernel update never arrives without
  its matching `linux-modules-nvidia-*` package (otherwise the GPU, and any monitors wired to it,
  disappear after a reboot).
- **WSL:** **don't install any Linux NVIDIA driver.** The Windows driver provides `/usr/lib/wsl/lib/`.
  `nvidia-smi` should already work.
- **AMD:** ROCm (check AMD's support matrix for this Ubuntu release and GPU before installing
  anything; ask). **Intel Arc:** Intel's oneAPI/XPU PyTorch builds.
- No GPU → CPU builds only.

## Step 2 — Workspace with uv

```bash
mkdir -p ~/dev/ml-sandbox && cd ~/dev/ml-sandbox
[ -f pyproject.toml ] || uv init --name ml-sandbox
uv add numpy pandas polars scipy scikit-learn matplotlib seaborn plotly jupyterlab ipykernel \
       statsmodels xgboost lightgbm pyarrow duckdb tqdm python-dotenv
uv run python -m ipykernel install --user --name ml-sandbox --display-name "Python (ml-sandbox)"
uv tool install jupyterlab
```

## Step 3 — PyTorch (latest stable)

Get the exact command from **https://pytorch.org/get-started/locally/** (Stable → Linux → Pip →
CUDA version ≤ the one `nvidia-smi` shows, or CPU):
```bash
TORCH_VARIANT=cpu      # set to the selector's value, e.g. cu130, when using an NVIDIA GPU
uv add torch torchvision torchaudio --index "https://download.pytorch.org/whl/$TORCH_VARIANT"
uv run python -c "import torch;print(torch.__version__, torch.cuda.is_available())"
```
The wheels bundle the CUDA runtime, so you don't need the full CUDA toolkit for PyTorch.

## Step 4 — TensorFlow / JAX

```bash
uv add "tensorflow[and-cuda]"   # GPU on Linux/WSL (CPU-only: uv add tensorflow)
uv run python -c "import tensorflow as tf;print(tf.__version__, tf.config.list_physical_devices('GPU'))"
# JAX: use the newest CUDA extra listed at https://docs.jax.dev/en/latest/installation.html, e.g.
# uv add "jax[cuda12]"
```
TensorFlow and PyTorch can pull conflicting CUDA wheels. Keep them in **separate uv projects** if
that happens.

## Step 5 — CUDA Toolkit (only if compiling CUDA code)

Use NVIDIA's official network repo for this Ubuntu version (or the `wsl-ubuntu` repo on WSL), from
<https://developer.nvidia.com/cuda-downloads>. Install the **newest** `cuda-toolkit` the driver
supports. On WSL, install **only** `cuda-toolkit`, never `cuda` or `cuda-drivers`.

## Step 6 — Optional tools

| Tool | Install |
|---|---|
| Miniforge (conda-forge) | `curl -fsSLo /tmp/mf.sh https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-$(uname -m).sh && bash /tmp/mf.sh -b -p ~/miniforge3` |
| Ollama | `curl -fsSL https://ollama.com/install.sh -o /tmp/ollama.sh && sh /tmp/ollama.sh` |
| R + RStudio | `apt_install r-base` + RStudio .deb from posit.co |
| DuckDB CLI | `mise use -g duckdb@latest` |
| Hugging Face CLI | `uv tool install "huggingface_hub[cli]"` |
| NVIDIA Container Toolkit (GPU in Docker) | NVIDIA's repo, see module 09 |

## Verify

```bash
cd ~/dev/ml-sandbox && uv run python -c "import numpy,pandas,sklearn;print('DS OK')"; nvidia-smi -L 2>/dev/null
```
