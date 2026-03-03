# CLAUDE.md — homelab-k8s-infra

This repository manages **shared Kubernetes infrastructure services** for the homelab cluster.
It must be deployed before `homelab-k8s-apps`.

## Repository Role

Provides cluster-wide infrastructure that applications depend on:
- **Traefik** — ingress controller (namespace: `traefik`)
- **cert-manager** — automatic TLS certificates via Let's Encrypt (namespace: `cert-manager`)
- **metrics-server** — cluster metrics (namespace: `kube-system`)
- **StorageClass** — `local-nas` pointing to NAS mount at `/mnt/nas/k8s`

**Deployment order:** homelab-base → **homelab-k8s-infra** → homelab-k8s-apps

## Cluster Context

```
Distribution:  k3s (single-node, Traefik and servicelb disabled at install time)
Node:          hestia (AMD E2-9000E, 16GB RAM)
Storage:       local NVMe (local-path) + NAS-backed (local-nas)
Domain:        .hestia suffix for local services; dynamic DNS for public ones
```

## Deployment

```bash
helmfile sync        # deploy / reconcile everything
helmfile diff        # preview changes before applying
helmfile destroy     # tear down (use with care)
```

Helmfile uses **manual execution** — there is no automatic reconciliation (GitOps was explicitly rejected, see ADR-0004 in homelab-base).

## Secrets Management

Secrets use **SOPS + age** encryption (see ADR-0007 in homelab-base).

- Plaintext: `secrets/*.yaml` — **never commit**, already in `.gitignore`
- Encrypted: `secrets/*.sops.yaml` — committed to Git
- Decrypt key file: `~/.config/age/key.txt` (set `SOPS_AGE_KEY_FILE`)

```bash
sops -e secrets/my-secret.yaml > secrets/my-secret.sops.yaml   # encrypt
sops -d secrets/my-secret.sops.yaml > secrets/my-secret.yaml   # decrypt
```

age public keys (both must be in `.sops.yaml` `creation_rules`):
- StarryNight: `age104xl76udz4syr3x4ltju2rcdd0mkvran3jv4520lyewgy2r0lgss9xly5z`
- hestia: `age14jnakt2vsmd37czh5wgn7ghkr276papa0vx2algs2lwhu9t3ufzsav9mlk`

## StorageClasses

| Class | Provisioner | Backing | Reclaim |
|---|---|---|---|
| `local-nas` | rancher.io/local-path | `/mnt/nas/k8s` (NFS) | Retain |
| `local-path` | rancher.io/local-path | Node-local NVMe | Delete |

Use `local-nas` for application data that needs persistence across pod restarts.
Use `local-path` for ephemeral or cache data.

## Naming & Namespace Conventions

- `traefik` — ingress controller
- `cert-manager` — certificate management
- `kube-system` — system components (metrics-server)
- TLS secrets: named `{service}-tls`
- Ingress hosts: `{service}.hestia` for local, dynamic DNS hostname for public

## What NOT to Do

- Do not enable k3s built-in Traefik or servicelb (they were disabled at install, see ADR-0003)
- Do not commit plaintext secret files
- Do not change the StorageClass reclaim policy from `Retain` without understanding data loss implications
- Do not add GitOps tooling (FluxCD, ArgoCD) without revisiting ADR-0004
