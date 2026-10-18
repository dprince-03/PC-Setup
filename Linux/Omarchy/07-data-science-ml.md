# 07 — Data Science / ML (Omarchy)

**Profiles:** data-ml

1. `omarchy install dev-env python` (02) gives you mise Python + uv.
2. GPU: Omarchy configures NVIDIA at install time, so check `nvidia-smi`. CUDA toolkit only if
   compiling: `pkg cuda`.
3. Workspace, PyTorch, TensorFlow, JAX: identical to `Linux/Ubuntu/07-data-science-ml.md` Steps 2–4.
4. GPU in Docker: `pkg nvidia-container-toolkit && sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker`.
5. Optional: `pkg ollama-cuda` (NVIDIA) / `ollama`; Omarchy also ships AI CLI installers
   (`omarchy-install-ai-claude`, etc.). Ask.

## Verify

`cd ~/dev/ml-sandbox && uv run python -c "import torch;print(torch.cuda.is_available())"`
