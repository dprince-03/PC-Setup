# 09 — DevOps and Cloud (Fedora)

**Profiles:** devops-cloud

## Step 1 — Container engine (ask: Podman, Docker CE, or both)

Fedora includes **Podman** (rootless, daemonless, Docker-CLI compatible). Many users need
real Docker for Compose/Desktop-era tooling:

```bash
# Conflicts: list them, then remove only with consent (Rule 6)
rpm -qa | grep -E '^(moby-|podman-docker|docker-)' || echo none
# sudo dnf remove -y moby-engine moby-cli moby-buildx moby-compose moby-containerd moby-runc podman-docker
add_repo_file https://download.docker.com/linux/fedora/docker-ce.repo
dnf_install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
# Non-root (root-equivalent; explain, then ask):  sudo usermod -aG docker "$USER"
```
If Docker's repo doesn't support this Fedora release yet (new release week), the `dnf` metadata
fetch returns 404. Use Podman in the meantime and tell the user (Rule 4b).
Podman extras: `dnf_install podman-compose podman-docker` (the `podman-docker` shim only if
Docker CE is **not** installed).

## Step 2 — K8s / IaC / CI tools via mise (newest stable)

```bash
mise use -g kubectl@latest helm@latest kind@latest k9s@latest kubectx@latest minikube@latest \
            terraform@latest opentofu@latest act@latest hadolint@latest trivy@latest sops@latest age@latest
```
kind with Podman: `KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster`.
Fedora-native alternative for Terraform: HashiCorp's repo `https://rpm.releases.hashicorp.com/fedora/hashicorp.repo`.

## Step 3 — Ansible

`dnf_install ansible` (Fedora tracks it closely) or `uv tool install --with-executables-from ansible-core ansible`. Pick one.

## Step 4 — Cloud CLIs

```bash
# AWS v2: official installer (same as Ubuntu 09)
curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o /tmp/awscliv2.zip && unzip -qo /tmp/awscliv2.zip -d /tmp && sudo /tmp/aws/install --update
# Azure CLI
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
printf '[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc\n' | sudo tee /etc/yum.repos.d/azure-cli.repo >/dev/null
dnf_install azure-cli
# Google Cloud CLI
printf '[google-cloud-cli]\nname=Google Cloud CLI\nbaseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el9-x86_64\nenabled=1\ngpgcheck=1\nrepo_gpgcheck=0\ngpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg\n' | sudo tee /etc/yum.repos.d/google-cloud-sdk.repo >/dev/null
dnf_install google-cloud-cli
```
(Google publishes EL repos that work on Fedora. Check the current repo URL in Google's install docs.)

## Verify

Same as Ubuntu 09 Verify, plus `podman --version`.
