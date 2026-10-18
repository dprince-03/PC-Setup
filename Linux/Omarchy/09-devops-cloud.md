# 09 — DevOps and Cloud (Omarchy)

**Profiles:** devops-cloud · **Docker is preinstalled** (`docker`, `docker-compose`, `docker-buildx`).

```bash
systemctl is-active docker && docker run --rm hello-world
# Omarchy has a helper for sudo-less Docker. Explain that the docker group is root-equivalent, then ask:
# omarchy-setup-security-sudoless-docker      (undo: omarchy-remove-security-sudoless-docker)
pkg kubectl helm k9s kubectx minikube terraform opentofu ansible ansible-lint trivy sops age
mise use -g kind@latest act@latest
pkg aws-cli-v2 2>/dev/null || pkg aws-cli; pkg azure-cli 2>/dev/null || aurpkg azure-cli-bin
aurpkg google-cloud-cli     # ask (AUR)
omarchy-install-service-tailscale 2>/dev/null   # optional overlay network (ask)
```
`ufw-docker` is configured by Omarchy. Published container ports stay governed by it, but still bind
dev services to 127.0.0.1.

## Verify

Same as Ubuntu 09 Verify.
