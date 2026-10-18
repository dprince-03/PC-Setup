# 07 — Data Science / ML / DL (Arch)

**Profiles:** data-ml

1. **GPU driver:** module 01 Step 7. CUDA toolkit (optional; only to compile CUDA): `pac cuda cudnn` (newest).
   AMD: `pac rocm-hip-sdk rocminfo` (check GPU support; ask).
2. **Workspace:** identical to `Linux/Ubuntu/07-data-science-ml.md` Steps 2–4 (uv project, PyTorch from
   the official index URL, TensorFlow `[and-cuda]`, JAX). These are distro-independent wheels.
   (Arch also packages `python-pytorch-cuda`, but keep ML libs in uv projects, not system Python.)
3. **GPU containers:** `pac nvidia-container-toolkit && sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker`.
4. **Optional:** `pac ollama` (or `ollama-cuda` / `ollama-rocm`) · `aur miniforge` (or the official
   installer) · `pac r` · `pac duckdb` · `uv tool install "huggingface_hub[cli]"`.

## Verify

`nvidia-smi -L; cd ~/dev/ml-sandbox && uv run python -c "import torch;print(torch.cuda.is_available())"`
