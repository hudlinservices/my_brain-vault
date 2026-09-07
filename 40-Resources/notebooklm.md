---
title: "NotebookLM"
category: "concept"
tags:
  - notebooklm
  - research
  - ingest
  - mcp
  - knowledge-management
date: "2026-09-07"
updated: "2026-09-07"
---
# NotebookLM

Google's AI notebook product, and the vault's **research and ingest backend**. This page explains what it is, how the vault uses it, and why it is currently the single point of failure blocking two daily automations.

## What It Is

NotebookLM works on a **notebook → source → query** model:

- A **notebook** is a container of **sources** — PDFs, web URLs, YouTube videos, or pasted text.
- Queries are answered **grounded in those sources only**, with citations back to the exact passage — unlike a general chat, it won't answer from training data.
- That grounding is the whole point for a knowledge base: research findings come with verifiable provenance.

## How the Vault Uses It

NotebookLM reaches Claude sessions through the **NotebookLM MCP server** (global MCP config in `~/.claude.json`; tools `mcp__notebooklm__*`: `notebook_create`, `source_add`, `notebook_query`, `source_list`, …). See [[40-Resources/model-context-protocol|Model Context Protocol]] for the protocol layer, and [[10-Projects/pythonslayers/OneClickLM|OneClickLM]] for the auth bridge that makes it work.

Two roles:

1. **Ingest Path B** — the persistent **"Vault Ingest" notebook** (`1ded6d40-a688-4ea3-adb0-5b549b96eaf2`). Roy drops documents into it from any device (phone, tablet); the ingest flow polls `source_list`, queries each new source for "key concepts, entities, claims, facts", and files the synthesis into wiki pages. Logged in [[90-System/ingest-log|Ingest Log]].
2. **Research backend for [[20-Skills/research/knowledge-scout|Knowledge Scout]]** — one persistent **"Knowledge Scout" notebook** used across all runs (sources accumulate; created on first use). Each approved gap gets 2–4 web sources and three queries (overview → connections → implications) before synthesis into a wiki page.

## The Auth Mechanism — OneClickLM

NotebookLM has no public API. The MCP server authenticates via **oneclicklm**, which reuses browser cookies (`npx oneclicklm login` writes them; the MCP server reads them). Consequence: **the whole pipeline depends on a session cookie**, and it expires silently.

## Single Point of Failure (status as of 2026-09-07)

The cookie died around **2026-06-22**. The daily crons on the [[10-Projects/hudlinservices/pi-fleet|Pi Fleet]] have failed every run since:

| Automation | Last working |
|------------|--------------|
| 3:07 AM auto-ingest (Vault Ingest) | 2026-06-22 (~2.5 months down) |
| 4:13 AM Knowledge Scout cron | 2026-06-22 — 7 consecutive failed runs |

The scout did not die with it: [[20-Skills/research/knowledge-scout|Knowledge Scout]] fell back to **WebSearch + WebFetch** as its primary research path (skill updated 2026-09-07; interactive runs still use NotebookLM when a human can re-auth). But ingest Path B is fully blocked — phone drops sit unprocessed.

**Repair:** Roy runs `npx oneclicklm login` (interactive, browser cookie handshake — cannot be automated by an agent). After login: verify with `source_list(notebook_id="1ded6d40")`, bump [[90-System/ingest-log|Ingest Log]] "Last checked", and move the auth gap to Researched in [[20-Skills/research/gap-priorities|Gap Priorities]].

## Connections

- [[40-Resources/model-context-protocol|Model Context Protocol]] — the protocol NotebookLM tools speak
- [[10-Projects/pythonslayers/OneClickLM|OneClickLM]] — the Python Slayers product that supplies the auth bridge
- [[20-Skills/research/knowledge-scout|Knowledge Scout]] — the main consumer (research) + its fallback strategy
- [[90-System/ingest-log|Ingest Log]] — records what Path B has processed
- [[10-Projects/hudlinservices/pi-fleet|Pi Fleet]] — hosts the crons that depend on this staying up
- [[40-Resources/unified-knowledge-graph|Unified Knowledge Graph]] — Knowledge Gap 5 tracks the outage
