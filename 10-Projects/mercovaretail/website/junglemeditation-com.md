---
name: "junglemeditation.com"
status: "active"
priority: "P3"
corporate_entity: "[[../_corporate-hub|Mercova Retail]]"
path: "/home/projects/mercovaretail/junglemeditation.com"
deadline: ""
goals:
  - "Jungle Meditation brand website — wellness, meditation, breath, destress"
tags:
  - project
  - mercovaretail
  - website
  - brand
  - wellness
created: "2026-05-18"
updated: "2026-09-07"
---

# junglemeditation.com

Meditation brand website for [[10-Projects/mercovaretail/junglemeditation|Jungle Meditation]] — wellness, meditation, breath, destress. Serves static assets from the DO Spaces `auron` bucket (`auron.hyperspeedfiles.com`); infra detail in [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment & Hosting Infrastructure]].

## Deployment

- **Cluster:** DOKS, operated by [[10-Projects/hudlinservices/_corporate-hub|Hudlin Services]]
- **Image:** `registry.digitalocean.com/auron/junglemeditation:5.0.12` — self-boots via `/entrypoint.sh` using `JMN_*` vars from `junglemeditation-secrets` (migrates, then gunicorn on 127.0.0.1:8000 + nginx on 8102)
- **Static assets:** DO Spaces `auron` bucket via `auron.hyperspeedfiles.com`

## Notes

### 2026-09-07
- Registry repair (session HUD-KUBE): image moved to `:5.0.12`; the stale `command` override (old `gunicorn meditation.wsgi` + MED_* env bridge) was removed so the image CMD runs. Full detail in [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment & Hosting Infrastructure]].
- Scout #14: fixed stale Versa Retail frontmatter → Mercova Retail.

### 2026-09-01 — asset outage, recovered
- `auron` bucket found empty → every asset URL 403, homepage rendered unstyled.
- 618 objects re-uploaded from the pod's baked local copy (`/usr/share/nginx/html`, 241 MB); 46/52 homepage asset URLs verified 200.
- 5 images unrecoverable (3 article-card webp, 2 slider backgrounds).
- **Open:** rotate the SMTP password present in the pod env; bucket risk audit (other buckets also deny anonymous reads); nginx self-host hardening option.

### 2026-05-18
- Initial hub created from vault corporate alignment
