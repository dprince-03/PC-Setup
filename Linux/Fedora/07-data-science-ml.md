# 07 — Data Science / ML / DL (Fedora)

**Profiles:** data-ml

1. **GPU driver:** NVIDIA → module 01 Step 7 (`akmod-nvidia` + `xorg-x11-drv-nvidia-cuda`, which
   provides `nvidia-smi`). AMD → ROCm (Fedora packages ROCm natively: `dnf_install rocm-hip rocminfo`.
   Check GPU support first and ask). WSL → no Linux driver.
2. **Workspace:** identical to `Linux/Ubuntu/07-data-science-ml.md` Steps 2–4 (uv project
   `~/dev/ml-sandbox`, PyTorch via the selector index URL, TensorFlow `[and-cuda]`, JAX CUDA
   extra). These are pip wheels, so they don't depend on the distro.
3. **CUDA Toolkit (only to compile CUDA):** NVIDIA's official repo for the newest Fedora it
   supports (<https://developer.nvidia.com/cuda-downloads> → Linux → x86_64 → Fedora). Don't
   install `cuda-drivers` from NVIDIA's repo when RPM Fusion's akmod is in use (it conflicts). Install
   `cuda-toolkit` only. NVIDIA often lags a Fedora release behind, so tell the user if that's the case.
4. **Containers with GPU:** `nvidia-container-toolkit` from NVIDIA's repo, then
   `sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml`. Podman:
   `podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi`.
5. **Optional:** Miniforge, Ollama, R (`dnf_install R`), DuckDB (`mise use -g duckdb@latest`), as in
   the Ubuntu table.

## Verify

`cd ~/dev/ml-sandbox && uv run python -c "import torch;print(torch.__version__, torch.cuda.is_available())"`
