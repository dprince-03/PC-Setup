# 09 — DevOps and Cloud (Arch)

**Profiles:** devops-cloud · Most tools are in the official repos at their newest version.

```bash
# Containers (pick Docker or Podman; both is fine if the user wants it)
pac docker docker-compose docker-buildx
sudo systemctl enable --now docker.socket
# non-root (root-equivalent; ask): sudo usermod -aG docker "$USER"
# pac podman podman-compose    # alternative

# Kubernetes + IaC
pac kubectl helm k9s kubectx minikube terraform opentofu ansible ansible-lint
aur kind-bin 2>/dev/null || mise use -g kind@latest     # check the official repos first: pacman -Si kind
# CI / supply chain (ask)
pac trivy sops age hadolint-bin 2>/dev/null; mise use -g act@latest

# Cloud CLIs (ask which)
pac aws-cli-v2 2>/dev/null || pac aws-cli    # check that it's v2: aws --version
pac azure-cli 2>/dev/null || aur azure-cli-bin
aur google-cloud-cli                          # AUR (official tarball repackaged); ask
```
If a package name isn't found, run `pacman -Ss <name>` and then the AUR. Prefer official repos, then
vendor `-bin` packages, then mise.

## Verify

Same as Ubuntu 09 Verify.
