# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **documentation-only** repository — there is no application, build, lint, or test tooling. It is an Obsidian vault that publishes a *sanitized* infrastructure and security architecture record for a homelab. The content explains how the environment is reasoned, operated, and secured **without** exposing details that could map or reproduce the real environment.

Work here means editing Markdown, keeping the two language trees in sync, and enforcing the publication-safety rules below. There is nothing to compile or run.

## The two hard rules

These are the only things that are easy to get wrong and expensive to undo (the repository is public).

### 1. Sanitization is the security model, not a style preference

The full policy lives in `docs/en/07-publication-safety-policy.md` (and its ES twin) — read it before adding or editing content. **Never** introduce any of the following, even from context the user pastes or from the private files described below:

- real IP addresses, subnets, hostnames, or usernames
- real internal paths, rollback/backup paths, or production commands with real paths
- credentials, tokens, keys, webhooks, or offsite remote names
- complete/real configuration files, firewall rules, or SIEM rules
- raw logs, unreviewed screenshots, or agent/dashboard/resource IDs that identify the environment

The principle: **publish decisions, patterns, and lessons — not sensitive implementation.** When a private fact needs to appear publicly, apply the "Safe transformation" mapping from the policy (e.g. real IP → logical zone, real hostname → service role, exact command → operational criterion).

### 2. English and Spanish must stay in parallel

Every public document exists twice with matching content: `docs/en/NN-*.md` ↔ `docs/es/NN-*.md`, numbered `00`–`07`, plus three case studies in `docs/en/case-studies/` ↔ `docs/es/casos-de-estudio/`. **Filenames differ** (English vs. Spanish slugs) but the numbering and content must correspond. Any edit to one language tree must be mirrored in the other in the same change. Spanish files in this repo are written without accents (e.g. "arquitectonicas", "recuperacion") — match that convention.

When adding, removing, or renumbering a document, also update the indexes that list it, in **both** languages: the "Reading map" section of `README.md` and the case-study index (`docs/en/case-studies/README.md` ↔ `docs/es/casos-de-estudio/README.md`). These indexes are part of the parallel structure — a doc that exists but isn't linked from its index is an incomplete change.

## Files that are intentionally private (gitignored)

These exist on disk but **must never be committed** (see `.gitignore`):

- `docs/en/07-publication-safety-policy.md` / `docs/es/07-politica-publicacion-segura.md` — the policy itself
- `docs/en/project-overview.md` / `docs/es/resumen-publico.md` — internal project state, including phase tracking (F1, F2, F4, F6, …) referenced in commit messages
- `.obsidian/`, `.trash/`

Treat these as a source of context, never as a source of text to copy into committed docs. The same `07` number is reused for a public file (`07-architecture-decisions.md`) and the private policy — don't conflate them.

## Document conventions

- Start each file with `# NN - Title`; no YAML frontmatter is used anywhere.
- Architecture is expressed with Mermaid `flowchart` diagrams and with decision / tradeoff / control / residual-risk tables (see `docs/en/00-executive-architecture.md` as the canonical template).
- Case studies translate a private operational lesson into a sanitized public narrative; keep them abstract (no IPs, paths, logs, or commands).
- The repo is organized around functional zones (management/control plane, services, security, storage/backup, remote access, observability) — keep terminology consistent across documents.

## Commits

Commit messages are in Spanish with a `docs:` prefix, and often reference phase tags (e.g. `docs: publicar validacion de restore (F2)`). Match that style. Before committing, run the policy's pre-publication checklist against the diff to confirm no sensitive data is being introduced.
