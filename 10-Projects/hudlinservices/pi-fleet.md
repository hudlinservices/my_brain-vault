---
name: "Pi Fleet"
status: "active"
priority: "P2"
corporate_entity: "[[_corporate-hub|Hudlin Services]]"
path: "~/.ssh/config + ~/.claude/skills/pi-fleet/SKILL.md (fleet registry)"
goals:
  - "Run self-hosted services (Plex, SickChill, Transmission, Cockpit) on Raspberry Pis"
  - "Expose services securely via Cloudflare Tunnel — no open inbound ports"
  - "Host the daily knowledge-automation crons (3:07 AM ingest, 4:13 AM scout)"
tags:
  - project
  - infrastructure
  - raspberry-pi
  - self-hosting
  - cloudflare
created: "2026-09-07"
updated: "2026-09-07"
---

# Pi Fleet

[[10-Projects/hudlinservices/_corporate-hub|Hudlin Services]]' fleet of Raspberry Pis — the self-hosted half of the infrastructure stack. Where [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment]] covers DOKS containers, the Pi fleet runs media, automation, and the daily knowledge pipeline on hardware in the LAN.

## Fleet Inventory

The canonical registry is `~/.ssh/config` + the `/pi-fleet` skill (`~/.claude/skills/pi-fleet/SKILL.md`). Both must be updated when adding a Pi.

| Hostname | IP | User | Root | Services |
|----------|----|------|------|----------|
| plex1 | 10.1.14.100 | dev66 | ✓ | Plex Media Server, Cloudflared |
| plex2 | 10.1.14.101 | dev66 | ✓ | Claude Code 24/7 agent session |
| sickchill | 10.20.0.5 | dev66 | pending | SickChill |
| transmission | 10.20.0.11 | dev66 | ✓ | Transmission |
| transmission2 | 10.20.0.12 | dev66 | ✓ | Transmission |

> **Naming note (reconciled 2026-09-07):** the vault's shorthand "t2" in the global CLAUDE.md Active Work section is **transmission2** (10.20.0.12) — one fleet, five Pis. Earlier pages also rendered it "transmission2"; no separate `t2` host exists.

Two address ranges appear: plex1/plex2 on `10.1.14.x`, sickchill/transmission on `10.20.0.x` — subnet segmentation consistent with the VLAN model in [[40-Resources/network-architecture|Network Architecture]] (UDM-Pro gateway).

## Exposure — Cloudflare Tunnel

Every Pi is reachable remotely through **Cloudflare Tunnel** (`cloudflared`), never port-forwarding:

- **plex1** runs Plex behind a tunnel — the classic pain point was Plex server settings refusing to show through the tunnel (IPv6 quirks; `Preferences.xml` on Ubuntu). Admin is via Cockpit, not SSH-first.
- **Cockpit** is exposed at `cockpit.hudlincloud.com` with a Cloudflare Access gate (tunnel `86b2db60-…`, credentials-file `/home/dev66/.cloudflared/…`, ingress → `https://localhost:9090` with `noTLSVerify: true`).
- The tunnel model means no open inbound ports — same pattern documented for the cloud side in [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment & Hosting Infrastructure]].

## Admin & Maintenance

- **Cockpit** (web admin) is the day-to-day management surface for the Plex Pi.
- **Root access** on all Pis except sickchill ("pending") — `/pi-fleet update` runs `apt update && apt upgrade && apt autoremove` fleet-wide and flags Pis needing manual attention.
- **`/pi-fleet status`** reports hostname | IP | uptime | load | memory | disk | services per Pi.
- Fleet onboarding (`/pi-fleet add`) = add to SSH config + skill registry, paste the authorized-keys one-liner, verify dev66 + root SSH, first `apt upgrade`, set hostname.

## Daily Automation (crons on the fleet)

The fleet hosts the knowledge pipeline (per global CLAUDE.md):

| Time | Job |
|------|-----|
| **3:07 AM** | Auto-ingest from the [[40-Resources/notebooklm|NotebookLM]] Vault Ingest notebook → [[90-System/ingest-log|Ingest Log]] |
| **4:13 AM** | [[20-Skills/research/knowledge-scout|Knowledge Scout]] gap scan |

plex2 runs the Claude Code 24/7 agent session — the resident process behind these crons. The pipeline has been **down since 2026-06-22** (NotebookLM auth dead; needs `npx oneclicklm login`).

## Network Context

- **Gateway:** UniFi UDM Pro ([[40-Resources/network-architecture|Network Architecture]])
- **WAN:** Starlink — Cloudflare Tunnel matters precisely because CGNAT blocks inbound ports (see [[40-Resources/costa-rica|Costa Rica]])
- **Home automation:** fleet conversations cover Home Assistant Z-stick setup and VPN-segmented torrenting (see [[40-Resources/home-automation-stack|Home Automation Stack]])

## Connections

- [[10-Projects/hudlinservices/_corporate-hub|Hudlin Services]] — operates the fleet (the "hosts it" arm)
- [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment & Hosting Infrastructure]] — the cloud half; same Cloudflare Tunnel pattern
- [[40-Resources/network-architecture|Network Architecture]] — the LAN/VLAN layer the Pis live on
- [[40-Resources/unified-knowledge-graph|Unified Knowledge Graph]] — fleet listed under Automation

## Sources

- `/pi-fleet` skill — `~/.claude/skills/pi-fleet/SKILL.md` (fleet registry, workflows, guardrails)
- Vault ChatGPT-export conversations ([[chatgpt-conversations/hudlin-services/|Hudlin Services]]): *Plex Cloudflared Tunnel Setup* (122 msgs), *Cockpit setup with cloudflared* (243 msgs), *Check Pi RAM size* (87 msgs), *Stop transmission-daemon command*, *Plex nginx cloudflared*, *OpenVPN torrent setup*, *Home Assistant Z-stick setup*, *Setting static IPs*, *Starlink and Dream Wall setup*
