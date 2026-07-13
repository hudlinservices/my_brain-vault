---
title: "React Frontend Patterns"
category: "reference"
tags:
  - react
  - frontend
  - components
  - spa
  - design-systems
date: "2026-07-12"
source: "react.dev, NotebookLM Knowledge Scout research"
---
# React Frontend Patterns

How Fortisyn builds frontends. React powers every user-facing interface across the group — from e-commerce storefronts to smart home dashboards. Built by [[10-Projects/pythonslayers/_corporate-hub|Python Slayers]], designed by [[10-Projects/auronmedia/_corporate-hub|Auron Media]], hosted on [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment Infrastructure]].

## Where React Is Used

| Project | Entity | Type | Routes |
|---------|--------|------|--------|
| [[10-Projects/soleriatechnology/website/soleriatechnology-com|solariatechnology.com]] | Soleria Technology | React SPA | 12 |
| [[10-Projects/mercovaretail/versa-store|Versa Store]] | Mercova Retail | React SPA | Storefront |
| [[10-Projects/mercovaretail/versa-admin|Versa Admin]] | Mercova Retail | React SPA | Admin dashboard |
| [[10-Projects/pythonslayers/slf-for-roy|SLF for ROY]] | Python Slayers | React PWA | Personal companion |
| [[10-Projects/auronmedia/_corporate-hub|auron.media]] | Auron Media | React SPA | Agency site |

## Component Architecture

React applications are built from **components** — independent, reusable pieces of UI. A component can be a button, a form, or an entire page.

### Component Hierarchy
Break every UI into a tree:
```
Page
├── Header
│   ├── Logo
│   └── Navigation
├── HeroSection
│   ├── Heading
│   └── CTAButton
├── ProductGrid
│   └── ProductCard (×N)
│       ├── ProductImage
│       └── PriceTag
└── Footer
```

### Single Responsibility
Each component does **one thing**. If a component grows too complex, split it into smaller subcomponents.

### Data Flow — One Direction
Data flows **downward** from parent to child via **props**. Children never modify their parent's data directly.

## State Management

| Approach | When to Use | Fortisyn Context |
|----------|-------------|-----------------|
| **useState** | Data in one component | Form inputs, toggle states, UI flags |
| **Lift state up** | Siblings need same data | Product list + cart in storefront |
| **Context** | Many distant components need data | Current user, theme, cart contents |
| **Zustand** | Context gets unwieldy | Complex admin dashboards |
| **Redux** | Full-scale state machine | Overkill for current Fortisyn needs |

## SPA vs PWA

| | SPA (Single Page App) | PWA (Progressive Web App) |
|---|---|---|
| **Navigation** | Client-side routing, no page reloads | Same as SPA |
| **Offline** | No | Yes (service worker caching) |
| **Install** | No | Yes (add to home screen) |
| **Fortisyn use** | solariatechnology.com, auron.media, Mercova store/admin | SLF for ROY (personal companion needs offline) |

## Sharing Patterns Across Projects

### Reusable Component Library
Components used across projects should be extracted:
- **UI primitives** — buttons, inputs, modals
- **Brand elements** — logos, color tokens, typography (from [[10-Projects/auronmedia/signatures|Signatures]])
- **Layout components** — consistent page shells across entity sites

### Design Tokens
CSS custom properties for brand consistency:
```css
:root {
  --color-primary: #2d3436;
  --color-accent: #0984e3;
  --font-heading: 'Inter', sans-serif;
  --font-body: 'Source Sans Pro', sans-serif;
}
```

### DRY State
Find the **minimal representation of state**. Don't store derived values — compute them. If `cartTotal` is always `items.reduce(sum)`, don't store it separately.

## Build & Deploy

- All React apps containerized via Docker
- Deployed on [[40-Resources/cloud-deployment-infrastructure|Kubernetes (DOKS)]] as Deployments
- Served through Cloudflare Tunnel
- CI/CD pipeline: Git push → build → push to registry → kubectl apply

## Related Pages

- [[40-Resources/cloud-deployment-infrastructure|Cloud Deployment & Hosting]] — where React apps run
- [[40-Resources/django-platform-architecture|Django Platform Architecture]] — the API backend React apps talk to
- [[10-Projects/auronmedia/signatures|Signatures]] — brand identity system (design tokens, logos)
- [[10-Projects/pythonslayers/slf-for-roy|SLF for ROY]] — PWA example
- [[10-Projects/soleriatechnology/website/soleriatechnology-com|solariatechnology.com]] — SPA example (12 routes)
