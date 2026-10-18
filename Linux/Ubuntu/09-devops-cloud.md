# 09 — DevOps and Cloud (Ubuntu)

**Profiles:** devops-cloud

---

## Step 1 — Docker Engine (official Docker repo)

**On WSL with Docker Desktop's WSL integration: skip this step** (pick one engine).

```bash
# Remove conflicting distro packages ONLY after asking (Rule 6). List first:
dpkg -l | grep -E 'docker.io|docker-doc|docker-compose|podman-docker|containerd|runc' || echo "no conflicts"
# sudo apt remove docker.io docker-doc docker-compose podman-docker containerd runc   # if the user agrees

repo_supports https://download.docker.com/linux/ubuntu && echo "Docker repo supports $UCODENAME" || echo "NOT YET: see Rule 4b"
add_apt_repo docker https://download.docker.com/linux/ubuntu/gpg \
  "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $UCODENAME stable"
sudo apt update
apt_install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```
Non-root access (ask; the `docker` group is root-equivalent, so explain that):
`sudo usermod -aG docker "$USER"`, then log out and back in (or `newgrp docker`).
Rootless mode is the safer alternative: `dockerd-rootless-setuptool.sh install`.
UFW note: Docker's published ports bypass UFW. That's one reason the compose files bind to `127.0.0.1`.

Alternative engine (ask): **Podman** `apt_install podman podman-compose`.

## Step 2 — Kubernetes, IaC and other CLIs via mise (newest stable)

```bash
mise use -g kubectl@latest helm@latest kind@latest k9s@latest kubectx@latest minikube@latest \
            terraform@latest opentofu@latest
mise use -g act@latest hadolint@latest trivy@latest sops@latest age@latest   # CI/supply-chain helpers (ask)
```
Prefer the vendor repo for any tool the user wants managed by apt instead: e.g. HashiCorp
`https://apt.releases.hashicorp.com`, Kubernetes `https://pkgs.k8s.io/core:/stable:/<latest minor>/deb/`
(get the minor from `curl -Ls https://dl.k8s.io/release/stable.txt`).

Local cluster (ask; pick one): `kind create cluster --name dev` or `minikube start --driver=docker`.

## Step 3 — Ansible

```bash
uv tool install --with-executables-from ansible-core ansible
ansible --version | head -1
uv tool install ansible-lint
```

## Step 4 — Cloud CLIs (ask which)

```bash
# AWS CLI v2 (official installer; the arch can be x86_64 or aarch64)
curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o /tmp/awscliv2.zip
unzip -qo /tmp/awscliv2.zip -d /tmp && sudo /tmp/aws/install --update
# Azure CLI (Microsoft's repo)
add_apt_repo azure-cli https://packages.microsoft.com/keys/microsoft.asc \
  "deb [arch=$(dpkg --print-architecture)] https://packages.microsoft.com/repos/azure-cli/ $UCODENAME main"
sudo apt update && apt_install azure-cli
# Google Cloud CLI (Google's repo)
add_apt_repo google-cloud https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  "deb https://packages.cloud.google.com/apt cloud-sdk main"
sudo apt update && apt_install google-cloud-cli
```
If the Microsoft repo doesn't have this codename yet, use `uv tool install azure-cli` instead.
Logins (`aws configure sso`, `az login`, `gcloud init`) are done **by the user**.

## Step 5 — GPU containers (if data-ml + NVIDIA)

NVIDIA Container Toolkit from NVIDIA's repo
(<https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html>), then
`sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker`.
Test: `docker run --rm --gpus all ubuntu nvidia-smi`.

## Verify

```bash
docker run --rm hello-world | grep -m1 Hello; docker compose version; kubectl version --client | head -1
helm version --short; terraform -version | head -1; tofu -version | head -1; ansible --version | head -1
aws --version; az version -o tsv 2>/dev/null | head -1; gcloud --version | head -1
```
