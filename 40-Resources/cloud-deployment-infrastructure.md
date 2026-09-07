---
title: "Cloud Deployment & Hosting Infrastructure"
category: "concept"
tags:
  - infrastructure
  - kubernetes
  - digitalocean
  - cloudflare
  - hosting
  - devops
source: "WebSearch + WebFetch — DigitalOcean DOKS docs, Cloudflare Tunnel guides (2026); grounded in vault ChatGPT-export conversations"
date: "2026-06-17"
updated: "2026-09-07"
---
# Cloud Deployment & Hosting Infrastructure

How Fortisyn ships software. [[10-Projects/pythonslayers/_corporate-hub|Python Slayers]] builds the applications, [[10-Projects/hudlinservices/_corporate-hub|Hudlin Services]] hosts and operates them. This page documents the **cloud** half of that backbone — container orchestration and secure ingress. The **LAN** half lives in [[40-Resources/network-architecture|Network Architecture]].

> Until now "Kubernetes clusters" appeared as a one-line bullet on the Hudlin Services hub and a single row in [[40-Resources/django-platform-architecture|Django Platform Architecture]] ("Hosting: Hudlin Services (Kubernetes clusters)") with no explanation of *how*. This page fills that gap.

## The Stack

| Layer | Choice | Role |
|-------|--------|------|
| **Orchestration** | Kubernetes | Container scheduling, scaling, self-healing across all client workloads |
| **Managed K8s provider** | DigitalOcean Kubernetes (DOKS) | Managed control plane — Hudlin Services operates clusters without running masters |
| **Cluster CLI** | `doctl` + `kubectl` + `helm` | Provision (doctl), operate (kubectl), package/deploy (helm) — driven from a local laptop |
| **Packaging** | Helm charts | Pre-configured bundles of K8s resources; one chart deploys a whole app |
| **App runtime** | Docker containers (Gunicorn for Django) | 12-factor apps, containerized, see [[40-Resources/django-platform-architecture|Django triad]] |
| **Secure ingress** | Cloudflare Tunnel (`cloudflared`) | Outbound-only encrypted tunnel — no open inbound ports, works behind CGNAT/NAT |
| **Isolation** | Namespaces + [[40-Resources/network-architecture|VLANs]] | One cluster, isolated workloads per entity/client |

## Container Orchestration — DOKS

DigitalOcean Kubernetes (DOKS) gives Hudlin Services a **managed control plane**: DigitalOcean runs the Kubernetes masters, Hudlin Services manages only the workloads. Clusters are provisioned via the DigitalOcean control panel, `doctl` (the DO CLI), or Terraform, then operated locally with `kubectl` and packaged with `helm`.

The 2026 operating playbook Fortisyn follows:

- **Namespaces** organize workloads — a clean fit for Fortisyn's multi-entity model (one cluster, isolated namespaces per entity/client).
- **Horizontal Pod Autoscaler (HPA)** scales pods on demand — relevant for [[10-Projects/mercovaretail/_corporate-hub|Mercova Retail]] storefronts during product launches.
- **Helm charts** package the repeating [[40-Resources/django-platform-architecture|Store→API→Admin triad]] so each new brand storefront deploys from the same chart.
- **Managed control plane** reduces operational burden — the team focuses on application development, not infrastructure plumbing.

> Vault evidence: the ChatGPT-export conversation *"Kubernetes Setup on DigitalOcean"* (300 messages, filed under [[chatgpt-conversations/python-slayers/|Python Slayers]]) shows the real workflow — `doctl auth init`, `kube`, `helm` driven from a local laptop. Related: *"CDN link issue Kubernetes"*, *"K8s pod shell access"*, *"Kubernetes setup on DigitalOcean"*.

## Secure Ingress — Cloudflare Tunnel

Fortisyn exposes services (and self-hosted boxes like the Plex media server on a Raspberry Pi 4) through **Cloudflare Tunnel** rather than port-forwarding. `cloudflared` establishes a secure, **outbound-only** connection to Cloudflare's edge:

- **No open inbound ports** — the origin stays hidden behind Cloudflare; nothing is exposed directly to the internet.
- **Works behind NAT / CGNAT / dynamic IP** — no port forwarding, no static IP, no DDNS. This matters for [[40-Resources/costa-rica|Costa Rica]] deployments, where [[40-Resources/network-architecture|Starlink]] hands out CGNAT addresses.
- **Zero-trust access** — identity-based, authenticated, auditable connections in front of every service.
- **Runs as a systemd service** — `cloudflared` starts on boot and reconnects automatically.

> Vault evidence: *"Plex Cloudflared Tunnel Setup"* (122 messages, [[chatgpt-conversations/hudlin-services/|Hudlin Services]]) — a Raspberry Pi 4 running Ubuntu + Plex behind a Cloudflared tunnel, administered via Cockpit. Related: *"Plex nginx cloudflared"*, *"Cockpit setup with cloudflared"*.

## Static Assets — DigitalOcean Spaces

Storefront media and static assets ship through **DigitalOcean Spaces** buckets (S3-compatible object storage) rather than from the app images. Buckets per brand: `auron` (served via `auron.hyperspeedfiles.com`), plus `mercova`, `fortisyn`, `slayers`, `hud-prod`, `versa`, `hudcdn`. [[10-Projects/mercovaretail/website/junglemeditation-com|junglemeditation.com]] loads its css/js/images from the `auron` bucket.

### 2026-09-01 incident — auron bucket wiped

The junglemeditation.com homepage rendered unstyled when every asset URL in the `auron` bucket returned 403 — `list_objects_v2` showed 0 objects and `style.css` was NoSuchKey. Recovery: re-uploaded **618 objects** from the pod's baked local copy (`/usr/share/nginx/html`, 241 MB) with public-read ACLs, using the app's own credentials — no app or image changes. Verified 46/52 homepage asset URLs return 200. 5 images unrecoverable (3 article-card webp, 2 slider backgrounds).

**Open action items:**
- 🔴 **Rotate the SMTP password** present in the junglemeditation pod environment.
- 🟠 **Bucket risk audit** — other buckets (`mercova`, `fortisyn`, `slayers`, `hud-prod`, `versa`, `hudcdn`) also deny anonymous reads and may be wiped the same way.
- 🟡 **Hardening option** — rebuild the image to self-host static via the pod's nginx (`/cs/ /js/ /im/ /media/` locations already exist) and drop the bucket dependency.

### 2026-09-07 registry repair (session HUD-KUBE)

DO container registry swept: 21 repos, every tag now resolves as a single image manifest (no OCI index + attestation multi-manifests). Three "pod restart = ImagePull" gaps fixed: fortisyn-website now pinned by tag `:5.0.1`, royhudlin prod moved `5.0.13` → `:5.0.17`, junglemeditation moved `1.0.1` → `:5.0.12` (its stale `command` override — old `gunicorn meditation.wsgi` + MED_* env bridge — was removed; the image self-boots via `/entrypoint.sh` from `JMN_*` vars in `junglemeditation-secrets`). Legacy `meditation` and `jungle-wear` repos deleted on Roy's word. Registry policy lives in the global `~/.claude/CLAUDE.md` (build with `--provenance=false --sbom=false`, explicit version tags, keep 3 latest). Also noted: hudlincloud.com returns 526 (origin SSL) while chat.hudlincloud.com is 200.

## Who Uses It

| Consumer | What runs here |
|----------|---------------|
| [[10-Projects/mercovaretail/_corporate-hub\|Mercova Retail]] | The Django [[40-Resources/django-platform-architecture\|Store→API→Admin triad]] + brand storefronts |
| [[10-Projects/pythonslayers/chat-assistant\|Chat Assistant]] / [[10-Projects/pythonslayers/spanish-tutor\|Spanish Tutor]] / [[10-Projects/pythonslayers/slf-for-roy\|SLF for ROY]] | Personal full-stack apps |
| [[10-Projects/innovatience/accounting\|Chart of Accounts Pro]] | Node.js consulting app |
| Self-hosted services (Plex, Cockpit) | Raspberry Pi behind Cloudflare Tunnel |

## Connections

- [[10-Projects/hudlinservices/_corporate-hub|Hudlin Services]] — operates the clusters and tunnels (the "hosts it" in the triad)
- [[10-Projects/pythonslayers/_corporate-hub|Python Slayers]] — builds the apps that deploy here (the "builds it")
- [[40-Resources/django-platform-architecture|Django Platform Architecture]] — the apps this infrastructure hosts
- [[40-Resources/network-architecture|Network Architecture]] — the on-premises/LAN counterpart (Ubiquiti/UniFi)
- [[40-Resources/costa-rica|Costa Rica]] — why outbound-only ingress matters (Starlink CGNAT)
- [[10-Projects/pythonslayers/OneClickLM|OneClickLM]] — another deployable Node/TS service in the portfolio

## Sources

- [DigitalOcean — How To Deploy a Scalable and Secure Django Application with Kubernetes](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-scalable-and-secure-django-application-with-kubernetes)
- [DigitalOcean — From Containers to Kubernetes with Django](https://www.digitalocean.com/community/tutorial-series/from-containers-to-kubernetes-with-django)
- [Devtron — How to Deploy Applications to Kubernetes on DigitalOcean](https://devtron.ai/blog/deploy-applications-to-kubernetes-digitalocean/)
- [Cloudflare Tunnel on Raspberry Pi: Zero-Trust Access Without Ports — Brian Haman, PhD](https://www.brianhaman.com/grc-blog/cloudflare-tunnel-raspberry-pi-zero-trust)
- [Expose a Self-Hosted App with Cloudflare Tunnel — No Open Ports (RDP.sh)](https://rdp.sh/en/blog/expose-a-self-hosted-app-with-cloudflare-tunnel)
- Vault ChatGPT-export conversations: *Kubernetes Setup on DigitalOcean*, *Plex Cloudflared Tunnel Setup*
</content>
</invoke>
