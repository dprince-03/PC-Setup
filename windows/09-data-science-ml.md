# 09 — Data Science, ML and Deep Learning (Windows)

**Profiles:** data-ml · **Admin:** yes · Depends on: 04 (Python + uv), 06 (VS Code/PyCharm),
03 (WSL2, for TensorFlow GPU and Linux-only libraries)

---

## Step 1 — GPU check

```powershell
Get-CimInstance Win32_VideoController | Select Name, DriverVersion
nvidia-smi 2>$null       # shows driver + max supported CUDA version, if an NVIDIA GPU + driver exist
```
- **NVIDIA GPU:** make sure the **latest NVIDIA Game Ready or Studio driver** is installed (ask
  which; Studio is the steadier choice for ML). Get it from NVIDIA's site or the NVIDIA App
  (`winget search "NVIDIA App"`; official publisher only). One Windows driver serves both Windows
  and WSL2. **Never install a Linux NVIDIA driver inside WSL.**
- **AMD / Intel GPU:** PyTorch on Windows can use DirectML (`torch-directml`) or Intel XPU builds.
  Tell the user it's experimental; CPU is the default.
- **No GPU:** install CPU builds only.

## Step 2 — Environment strategy (ask; default = uv)

| Approach | When |
|---|---|
| **uv projects** (default) | most ML/DS work: fast and reproducible (`pyproject.toml` + `uv.lock`) |
| **Miniforge (conda-forge)** | heavy non-Python binary deps (GDAL, some CUDA libs, R) |

```powershell
# Optional conda (Miniforge is the community distribution, no Anaconda licence issues):
Install-App 'CondaForge.Miniforge3'
# conda config --set auto_activate_base false
```
Don't install full Anaconda unless the user asks. It's heavy, and commercial use has licensing
terms.

## Step 3 — Base DS/ML workspace with uv

```powershell
$ws = "$env:USERPROFILE\dev\ml-sandbox"
New-Item -ItemType Directory -Force $ws | Out-Null; Set-Location $ws
uv init --no-workspace --name ml-sandbox 2>$null
uv add numpy pandas polars scipy scikit-learn matplotlib seaborn plotly jupyterlab ipykernel notebook `
       statsmodels xgboost lightgbm pyarrow duckdb tqdm python-dotenv
uv run python -m ipykernel install --user --name ml-sandbox --display-name "Python (ml-sandbox)"
```
User-wide notebook launcher: `uv tool install jupyterlab`.

## Step 4 — PyTorch (latest stable)

Get the right command from the official selector at **https://pytorch.org/get-started/locally/**
(choose Stable → Windows → Pip → Python → your CUDA version or CPU). It's always an index-URL install:
```powershell
# NVIDIA: pick the newest CUDA variant the selector lists that is ≤ the "CUDA Version" in nvidia-smi
uv add torch torchvision torchaudio --index https://download.pytorch.org/whl/<cuXXX>   # from the selector
# CPU only:
uv add torch torchvision torchaudio --index https://download.pytorch.org/whl/cpu
```
The pip wheels bundle the CUDA runtime, so **you don't need the full CUDA Toolkit** for PyTorch.

Verify:
```powershell
uv run python -c "import torch;print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'cpu')"
```

## Step 5 — TensorFlow / JAX

- **TensorFlow GPU isn't supported on native Windows** (it stopped after 2.10). For GPU use
  **WSL2**: in the Ubuntu WSL distro, `uv add "tensorflow[and-cuda]"` (see Linux/Ubuntu module 07).
  On native Windows, install CPU only: `uv add tensorflow`.
- **JAX:** GPU builds are Linux-only, so use WSL2 (`uv add "jax[cuda12]"`, or the newest CUDA extra
  in JAX's install docs). On Windows, CPU only: `uv add jax`.

## Step 6 — CUDA Toolkit (only if the user compiles CUDA code / custom kernels)

```powershell
Install-App 'Nvidia.CUDA'        # newest toolkit; needs VS 2026 C++ workload for nvcc host compiler
nvcc --version
```
Make sure the toolkit's CUDA major is supported by the installed driver (`nvidia-smi`, top right).

## Step 7 — Extra tools (ask)

| Tool | Install |
|---|---|
| Ollama (local LLMs) | `Install-App 'Ollama.Ollama'` |
| LM Studio | `winget search "LM Studio"` (official publisher: Element Labs) |
| R + RStudio | `Install-App 'RProject.R'`; `Install-App 'Posit.RStudio'` |
| DBeaver / DuckDB CLI | `DBeaver.DBeaver`; `DuckDB.cli` |
| Hugging Face CLI | `uv tool install "huggingface_hub[cli]"` |
| MLflow / Weights & Biases | per project: `uv add mlflow` / `uv add wandb` |
| Git LFS (for datasets/models) | already installed in module 02 |

## Verify

```powershell
uv run --directory $ws python -c "import numpy, pandas, sklearn, matplotlib; print('DS stack OK')"
uv run --directory $ws jupyter lab --version
nvidia-smi
```
