# 11 — DevOps and Cloud (Windows)

**Profiles:** devops-cloud · **Admin:** yes · **Reboot:** maybe (Docker Desktop + WSL) ·
Depends on: 03 (WSL2)

---

## Step 1 — Containers: Docker Desktop (or an alternative)

**Detect:** `docker version`, `docker context ls`, `winget list Docker.DockerDesktop`,
`winget list RedHat.Podman-Desktop`, `winget list SUSE.RancherDesktop`.

> **Licence note:** Docker Desktop is free for personal use, education, open source, and small
> businesses (<250 employees and <$10M revenue). Larger companies need a paid subscription. Tell
> the user. Free alternatives: **Podman Desktop** (`RedHat.Podman-Desktop`) or **Rancher Desktop**
> (`SUSE.RancherDesktop`). Install **one** container engine only.

```powershell
Install-App 'Docker.DockerDesktop' @('--override','install --quiet --accept-license --backend=wsl-2')
```
Afterwards: the user signs out and back in (or reboots), starts Docker Desktop, and enables
**Settings → Resources → WSL integration** for their distro. Then:
```powershell
docker run --rm hello-world
docker compose version; docker buildx version
```
If the user isn't an admin, add them to the `docker-users` group (ask):
`Add-LocalGroupMember -Group docker-users -Member $env:USERNAME`, then sign out and back in.

## Step 2 — Kubernetes tooling

| Tool | winget ID |
|---|---|
| kubectl | `Kubernetes.kubectl` |
| Helm | `Helm.Helm` |
| kind (K8s in Docker) | `Kubernetes.kind` |
| minikube (alternative local cluster) | `Kubernetes.minikube` |
| k9s (TUI) | `Derailed.k9s` |
| kubectx / kubens | `ahmetb.kubectx` (check with `winget search kubectx`) |
| krew (kubectl plugins) | `Kubernetes.krew` |
| Lens / Headlamp GUI (optional) | `winget search Headlamp` (Kubernetes SIG official) |

Docker Desktop can also run a single-node cluster: Settings → Kubernetes → Enable. Ask the
user to pick **one** local cluster tool (Docker Desktop K8s, kind, or minikube).

```powershell
'Kubernetes.kubectl','Helm.Helm','Kubernetes.kind','Derailed.k9s' | % { Install-App $_ }
kind create cluster --name dev   # only if the user wants a cluster now
kubectl cluster-info --context kind-dev
```

## Step 3 — Infrastructure as Code and config management

| Tool | Install | Note |
|---|---|---|
| Terraform | `Install-App 'Hashicorp.Terraform'` | BSL licence |
| OpenTofu | `Install-App 'OpenTofu.Tofu'` | open-source Terraform fork; ask which (or both) |
| Packer / Vagrant (optional) | `Hashicorp.Packer`, `Hashicorp.Vagrant` | Vagrant needs a hypervisor |
| Ansible | **inside WSL2 only** (control nodes don't run on Windows): `uv tool install --with-executables-from ansible-core ansible` in WSL | |
| Pulumi (optional) | `winget search Pulumi` | |

## Step 4 — Cloud CLIs (ask which clouds)

| Cloud | winget ID | Login (the user does this) |
|---|---|---|
| AWS CLI v2 | `Amazon.AWSCLI` | `aws configure sso` |
| Azure CLI | `Microsoft.AzureCLI` | `az login` |
| Google Cloud CLI | `Google.CloudSDK` | `gcloud init` |
| Azure Developer CLI (optional) | `Microsoft.Azd` | `azd auth login` |
| Azure Functions Core Tools (optional) | `Microsoft.Azure.FunctionsCoreTools` | |
| Oracle Cloud CLI (optional) | `Oracle.OCI-CLI` | |
| DigitalOcean doctl (optional) | `DigitalOcean.Doctl` | |
| Cloudflare Wrangler (optional) | `npm i -g wrangler` | |

Never run the login commands for the user. Credentials are theirs to enter.

## Step 5 — CI/CD and supply-chain helpers

| Tool | Install |
|---|---|
| GitHub CLI | done in module 02 |
| act (run GitHub Actions locally) | `Install-App 'nektos.act'` |
| Trivy (image/IaC scanning) | `winget search trivy` (official publisher: Aqua Security) |
| hadolint (Dockerfile linter) | `Install-App 'hadolint.hadolint'` |
| SOPS / age (secrets in git) | `Mozilla.SOPS`, `FiloSottile.age` |

## Verify

```powershell
docker version --format '{{.Server.Version}}'; kubectl version --client; helm version --short
terraform -version; tofu -version; aws --version; az version --output tsv 2>$null; gcloud --version
```
