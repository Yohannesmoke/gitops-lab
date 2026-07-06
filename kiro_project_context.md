# Kiro Project Context — GitOps Lab & Safaricom Observability Platform
 
This file gives Kiro full context about this project, what has been done,
what is in progress, and what the goals are. Load this at the start of every session.
 
---
 
## Who is the user
 
- Name: Yohannes Mekonen
- Role: Works at Safaricom Ethiopia (cloud/infrastructure team)
- GitHub: Yohannesmoke
- Local machine: Windows, Docker Desktop installed
- IDE: Kiro (VS Code-based)
- Skill level: Familiar with infrastructure concepts, learning Kubernetes/GitOps hands-on
 
---
 
## Project 1 — Production Safaricom Ethiopia Platform (Reference/Read-Only)
 
### Location
```
C:\Users\yohannes.mekonen\OneDrive - Safaricom Ethiopia\Documents\MY Config GRAFANA\
```
 
### What it is
A production-grade GitOps-driven Kubernetes observability and infrastructure platform
for Safaricom Ethiopia's two data centers (mdc1 and mdc2).
 
### Key documents already created in this folder
- `INFRASTRUCTURE_AND_CI_CD_GUIDE.md` — original project guide
- `CI_CD_PIPELINE_GUIDE.md` — CI/CD pipeline explanation
- `BLACKBOX_EXPORTER_CONFIGURATION_GUIDE.md` — blackbox exporter deep dive
- `GITOPS_DEEP_DIVE.md` — comprehensive explanation of every GitOps concept used
  (Helm, Kustomize, Flux, manifests, Kubernetes objects, ESO, cert-manager, Kyverno,
  Contour, Harbor, VictoriaMetrics, VictoriaLogs, Tempo, OTel, Grafana — all explained
  from first principles with real examples from the project files)
- `LOCAL_GITOPS_LAB_SETUP.md` — full step-by-step guide to build the local lab
- `KIRO_PROJECT_CONTEXT.md` — this file
 
### Production stack summary
| Layer | Technology |
|---|---|
| GitOps engine | Flux CD (OCIRepository source from Harbor) |
| Package manager | Helm (all charts pinned by version + SHA256 digest) |
| Config customization | Kustomize (overlays + variable substitution) |
| Container registry | Harbor (private OCI registry) |
| Policy engine | Kyverno |
| Certificate management | cert-manager + ADCS Issuer + trust-manager |
| Secret management | External Secrets Operator (ESO) pulling from vault |
| Ingress | Contour (Envoy-based) |
| Metrics storage | VictoriaMetrics Cluster (v1.126.0) |
| Log storage | VictoriaLogs Cluster (×2 instances, v1.33.0) |
| Trace storage | Grafana Tempo (v2.9.0, distributed) |
| Alerting | Alertmanager (v0.28.1) |
| Dashboards | Grafana (v12.0.2) via Grafana Operator |
| Telemetry pipeline | OpenTelemetry Collector (otel-gateway, 12 replicas) |
| K8s telemetry agent | Splunk OTel Collector (DaemonSet) |
| Infrastructure monitoring | iDRAC, PowerMax, PowerScale, Netscaler, AVI exporters |
| Storage | Dell CSM (Isilon/PowerScale NFS) |
 
### Key architectural patterns
- Multi-datacenter: `${datacenter}` variable substitution deploys same configs to mdc1 + mdc2
- Air-gapped: every image mirrored to internal Harbor, pinned by SHA256 digest
- Persistent OTel queues: 50Gi PVCs per collector pod so data survives restarts
- Zero secrets in Git: all credentials flow through ESO from external vault
- Strict deployment order via Flux `dependsOn` chain:
  `common → common-post-configs → common-extras → common-extras-post-configs → extras → extras-post-configs`
 
---
 
## Project 2 — Local GitOps Lab (Active — In Progress)
 
### Goal
Build a local mini version of the production stack on Docker Desktop Kubernetes
for learning and experimentation. Practice the full GitOps workflow.
 
### GitHub repository
- URL: https://github.com/Yohannesmoke/gitops-lab
- Local clone path: `C:\gitops-lab`
- Flux sync path: `clusters/local`
 
### Target local stack
| Component | Purpose | Local URL |
|---|---|---|
| Flux CD | GitOps engine — watches GitHub, applies changes | (internal) |
| Harbor | Private container registry | http://localhost:30080 |
| VictoriaMetrics | Metrics storage (single-node for local) | (internal) |
| VictoriaLogs | Log storage | (internal) |
| OTel Collector | Telemetry pipeline (receives + forwards) | (internal) |
| Grafana | Dashboards | http://localhost:30300 |
 
### Credentials for local lab
| Service | Username | Password |
|---|---|---|
| Harbor | admin | Harbor12345 |
| Grafana | admin | admin123 |
 
### Helm chart sources used
| Chart | Helm Repo URL |
|---|---|
| Harbor | https://helm.goharbor.io |
| VictoriaMetrics | https://victoriametrics.github.io/helm-charts |
| VictoriaLogs | https://victoriametrics.github.io/helm-charts |
| OTel Collector | https://open-telemetry.github.io/opentelemetry-helm-charts |
| Grafana | https://grafana.github.io/helm-charts |
 
### Folder structure (to create in gitops-lab repo)
```
gitops-lab/
└── clusters/
    └── local/
        ├── flux-system/              ← created by Flux bootstrap (do not touch)
        ├── infrastructure.yaml       ← Flux Kustomization pointing to infra folder
        └── infrastructure/
            ├── kustomization.yaml    ← lists all components
            ├── harbor/
            │   ├── namespace.yaml
            │   ├── helmrepo.yaml
            │   └── helmrelease.yaml
            ├── victoria-metrics/
            │   └── helmrelease.yaml
            ├── victoria-logs/
            │   └── helmrelease.yaml
            ├── otel/
            │   └── helmrelease.yaml
            └── grafana/
                └── helmrelease.yaml
```
 
### Current progress / what has been done
1. ✅ Flux CLI installed on Windows machine
2. ✅ GitHub repo `gitops-lab` created (https://github.com/Yohannesmoke/gitops-lab)
3. ✅ Flux bootstrap succeeded once (but cluster had TLS issues afterward)
4. ⚠️ Docker Desktop Kubernetes had TLS handshake timeout issue — likely caused by
   corporate network / VPN / proxy interference
5. ⏳ Infrastructure files (Harbor, VictoriaMetrics, VictoriaLogs, OTel, Grafana)
   have NOT been committed to the repo yet — this is the next step
 
### Known issues encountered
- **TLS handshake timeout** on `kubectl get pods`: Docker Desktop Kubernetes crashed
  or corporate network is intercepting TLS. Fix: Reset Kubernetes in Docker Desktop,
  try on personal hotspot / home network (not corporate VPN).
- **Authorization failed on bootstrap**: GitHub PAT had wrong scope. Fix: Use classic
  PAT with full `repo` scope.
- **Context deadline exceeded on bootstrap**: Corporate network proxy issue.
  Fix: Use personal hotspot.
 
---
 
## Next Steps (what to do when resuming on the new laptop)
 
### Step 1 — Verify prerequisites are installed
```cmd
kubectl version --client
helm version
flux --version
git --version
```
 
### Step 2 — Verify Docker Desktop Kubernetes is healthy
```cmd
kubectl get nodes
kubectl config current-context
```
Must show `docker-desktop` as context and node STATUS = `Ready`.
 
### Step 3 — Clone the repo
```cmd
cd C:\
git clone https://github.com/Yohannesmoke/gitops-lab.git
cd gitops-lab
```
 
### Step 4 — Check if Flux is already installed
```cmd
kubectl get pods -n flux-system
```
If pods are Running, skip to Step 6.
If namespace doesn't exist or pods are not running, do Step 5.
 
### Step 5 — Bootstrap Flux (if needed)
```cmd
flux bootstrap github ^
  --owner=Yohannesmoke ^
  --repository=gitops-lab ^
  --branch=main ^
  --path=clusters/local ^
  --personal ^
  --token-auth
```
Use a classic PAT with `repo` scope. Must end with `✔ all components are healthy`.
 
### Step 6 — Create infrastructure folder structure
```cmd
cd C:\gitops-lab
mkdir clusters\local\infrastructure\harbor
mkdir clusters\local\infrastructure\victoria-metrics
mkdir clusters\local\infrastructure\victoria-logs
mkdir clusters\local\infrastructure\otel
mkdir clusters\local\infrastructure\grafana
```
 
### Step 7 — Create all infrastructure files
All file contents are in `LOCAL_GITOPS_LAB_SETUP.md` (Phase 4).
Ask Kiro to create them: "Create all the infrastructure files for the gitops-lab"
 
### Step 8 — Commit and push
```cmd
cd C:\gitops-lab
git add .
git commit -m "add infrastructure stack"
git push
```
 
### Step 9 — Watch deployment
```cmd
flux get all -A --watch
kubectl get pods -n harbor
kubectl get pods -n victoria-metrics
kubectl get pods -n victoria-logs
kubectl get pods -n otel
kubectl get pods -n grafana
```
 
---
 
## How to Use Kiro Effectively on This Project
 
### Ask Kiro to create files
"Create the file clusters/local/infrastructure/harbor/helmrelease.yaml with the Harbor HelmRelease config"
 
### Ask Kiro to diagnose issues
Paste kubectl/flux output directly into chat. Example:
"I got this error: [paste output] — what does it mean and how do I fix it?"
 
### Ask Kiro to explain concepts
"Explain what a HelmRelease dependsOn does"
"Why does my pod show CrashLoopBackOff?"
 
### Ask Kiro to modify configs
"Change the VictoriaMetrics retention from 7d to 30d"
"Add a loki datasource to Grafana"
 
### Reference files Kiro should know about
- `#File GITOPS_DEEP_DIVE.md` — full explanation of all technologies
- `#File LOCAL_GITOPS_LAB_SETUP.md` — full setup guide with all file contents
- `#File KIRO_PROJECT_CONTEXT.md` — this context file
 
---
 
## Useful Commands Cheat Sheet
 
```cmd
# Verify cluster
kubectl get nodes
kubectl config current-context
 
# Check all Flux resources
flux get all -A
 
# Watch Flux reconcile live
flux get all -A --watch
 
# Force immediate sync (don't wait 1 min)
flux reconcile source git flux-system
flux reconcile kustomization infrastructure
 
# Check HelmReleases
flux get helmreleases -A
 
# Check why something failed
kubectl describe helmrelease grafana -n grafana
kubectl describe kustomization infrastructure -n flux-system
 
# Check Flux controller logs
kubectl logs -n flux-system deployment/helm-controller --tail=50
kubectl logs -n flux-system deployment/source-controller --tail=50
kubectl logs -n flux-system deployment/kustomize-controller --tail=50
 
# Check pods in all namespaces
kubectl get pods -A
 
# Check events for errors
kubectl get events -A --sort-by='.lastTimestamp'
 
# Port-forward if NodePort not working
kubectl port-forward svc/harbor -n harbor 8080:80
kubectl port-forward svc/grafana -n grafana 3000:80
```
 
---
 
## Troubleshooting Quick Reference
 
| Error | Cause | Fix |
|---|---|---|
| `TLS handshake timeout` | K8s crashed or VPN interference | Restart Docker Desktop or Reset Kubernetes. Try on hotspot. |
| `context deadline exceeded` | Corporate proxy blocking | Use personal hotspot, disconnect VPN |
| `authorization failed` on bootstrap | PAT wrong scope | Recreate classic PAT with full `repo` scope |
| Pods `Pending` | Not enough RAM/CPU | Docker Desktop → Resources → 8GB RAM, 4 CPU |
| HelmRelease `reconciling` forever | Can't download chart | Check network, try `flux logs --follow` |
| Harbor `CrashLoopBackOff` | Disk space | Increase disk in Docker Desktop resources |
| `Unable to connect to server` | Wrong kubectl context | Run `kubectl config use-context docker-desktop` |
| Bootstrap `component manifests are up to date` | Already bootstrapped | That's fine, continue to next step |
 
---
 
## Background Context for Kiro
 
- The user is building this lab to understand how the production Safaricom Ethiopia
  platform works, using the same tools (Flux, Helm, Kustomize, VictoriaMetrics,
  VictoriaLogs, OTel, Grafana, Harbor) at smaller scale locally.
- The production platform uses an OCI artifact source (Harbor) instead of direct
  Git source for Flux — the local lab uses direct GitHub source for simplicity.
- The production platform has multi-datacenter variable substitution — the local lab
  is single environment (no substitution needed).
- All conversations, concepts, and decisions from the original session are captured
  in the documents in:
  `C:\Users\yohannes.mekonen\OneDrive - Safaricom Ethiopia\Documents\MY Config GRAFANA\`
- The user prefers: direct explanations, step-by-step instructions, full file contents
  (not partial), and to understand WHY things work not just HOW.
 
GitHub - Yohannesmoke/gitops-lab
Contribute to Yohannesmoke/gitops-lab development by creating an account on GitHub.
 