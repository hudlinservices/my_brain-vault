---
name: "Roy Hudlin Store"
status: "active"
priority: "P3"
corporate_entity: "[[_corporate-hub|Mercova Retail]]"
path: "/home/projects/versaretail/royhudlin"
deadline: ""
goals:
  - "E-commerce storefront for Roy Hudlin brand merchandise and products"
tags:
  - project
  - mercovaretail
  - django
  - ecommerce
  - store
created: "2026-05-18"
updated: "2026-06-09"
---

# Roy Hudlin Store

E-commerce storefront for Roy Hudlin branded merchandise. Django application with media, logs, and custom config. Separate from the personal brand entity at [[10-Projects/royhudlin/_corporate-hub|Roy Hudlin]].

## Structure

```
royhudlin/
├── core/       ← Django app core
├── roy_config/ ← Personal site config
├── data/       ← Data/fixtures
├── logs/       ← Application logs
├── media/      ← Uploaded media
├── static/     ← Static assets
└── templates/  ← Django templates
```

## Notes

### 2026-09-07
- Scout #14: stale Versa Retail frontmatter fixed → Mercova Retail. On-disk path still `/home/projects/versaretail/` — rename pending (see [[10-Projects/mercovaretail/_corporate-hub|Mercova hub]]).

### 2026-05-18
- Initial hub created from vault corporate alignment
