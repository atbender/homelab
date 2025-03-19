# Homelab

This repository contains the infrastructure-as-code for my personal Kubernetes homelab. It uses GitOps principles with Flux CD to manage deployments and configurations across my home infrastructure.

## Overview

This homelab setup includes:

- **[Flux CD](https://fluxcd.io/)** for GitOps-based deployments
- **[Kustomize](https://kustomize.io/)** for Kubernetes resource templating
- **[SOPS](https://github.com/mozilla/sops)** with age for secret encryption
- **Monitoring stack** using Prometheus and Grafana
- **Self-hosted applications** like Linkding (bookmark manager)
- **Cloudflare Tunnels** for secure external access

## Repository Structure

```
├── apps/
│   ├── base/               # Base configurations for applications
│   │   └── linkding/       # Base Linkding configuration
│   └── staging/            # Environment-specific overlays
│       └── linkding/       # Staging-specific Linkding configs with Cloudflare tunnel
├── clusters/
│   └── staging/            # Staging cluster configuration
│       ├── flux-system/    # Flux bootstrap components
│       └── .sops.yaml      # SOPS encryption configuration
└── monitoring/
    └── controllers/        # Monitoring stack components
        ├── base/           # Base monitoring configuration
        │   └── kube-prometheus-stack/
        └── staging/        # Staging-specific monitoring config
```

## Applications

### Linkding

A self-hosted bookmark manager running in Kubernetes with:
- Persistent storage for bookmark data
- Encrypted secrets for credentials
- Cloudflare tunnel for secure external access without opening ports
- Accessible via my personal domain

## Getting Started

### Prerequisites

- A Kubernetes cluster
- [Flux CLI](https://fluxcd.io/docs/installation/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [SOPS](https://github.com/mozilla/sops)
- [age](https://github.com/FiloSottile/age) for encryption

### Bootstrap

1. Generate an age key for SOPS encryption:

```bash
age-keygen -o age.agekey
```

2. Create a Kubernetes secret with the age key:

```bash
cat age.agekey | kubectl create secret generic sops-age \
--namespace=flux-system \
--from-file=age.agekey=/dev/stdin
```

3. Bootstrap Flux on your cluster:

```bash
flux bootstrap github \
  --owner=your-github-username \
  --repository=homelab \
  --branch=main \
  --path=./clusters/staging \
  --personal
```

## Secret Management

This repository uses SOPS with age for secret encryption. Secrets are encrypted using the age public key defined in `.sops.yaml` and decrypted by Flux using the corresponding private key stored as a Kubernetes secret.

Example of encrypting a secret:

```bash
sops --encrypt --age age1gynmj9vwh8g2kmlgn0h7jhkkhjn92eyscjs6x8zysfd7jv34d4dqmk3n25 secret.yaml > encrypted-secret.yaml
```

## Monitoring

The monitoring stack is based on the kube-prometheus-stack Helm chart and includes:

- Prometheus for metrics collection
- Grafana for visualization (default admin password: xandi)
- AlertManager for alerting
- Various exporters for system and Kubernetes metrics

## Development Environment

This repository includes a devcontainer configuration for consistent development environments:

- Debian-based container
- Neovim editor
- ZSH as the default shell
- Port forwarding for local development (8080)

## Notes

- Remember to keep your `age.agekey` file secure and back it up safely
- The Grafana admin password is set in plain text in the Helm release values and should be changed for production use
- Cloudflare tunnel credentials are encrypted and stored as Kubernetes secrets

## License

This project is for personal use but feel free to use it as inspiration for your own homelab.
