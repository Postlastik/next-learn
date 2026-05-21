# Next.js Dashboard — BA workflow sandbox

This is `dashboard/final-example` from vercel/next-learn — a Next.js
App Router dashboard with invoices, customers, and authentication.
We use it as a sandbox to practice an AI-driven BA workflow.

Treat the existing application as a product we're extending.

## Tech stack

- Next.js (App Router) + React + TypeScript
- next-auth for authentication
- Postgres + bcrypt for data storage and password hashing
- Tailwind CSS for styling
- Zod for form validation

## Requirements workflow

Requirements live in `docs/requirements/`:
- `draft/` — work in progress, no approval needed yet
- `ready/` — approved by reviewers, ready for implementation

When the user describes a feature idea, improvement, or product change
and wants to formalize it as a requirement, use the `write-prd` skill
at `docs/skills/write-prd/SKILL.md`.

## Skills

Custom skills live in `docs/skills/`. Before taking actions a skill
covers, read its SKILL.md first:

- `docs/skills/write-prd/SKILL.md` — for writing Product Requirements
  Documents (PRDs).

## Git workflow

- Each new requirement gets its own branch: `prd/<short-slug>`.
- Open a draft PR (`gh pr create --draft`) while the requirement is
  in `draft/`.
- Move the markdown to `ready/` and mark the PR ready
  (`gh pr ready`) only when the requirement is approved.
- Commit messages follow Conventional Commits, e.g. "docs(prd): draft requirements for <feature name>".
- Promotion to ready uses a separate branch: "promote/<short-slug>".

## Conventions

- Requirement filenames: `YYYY-MM-DD-short-slug.md`
  (e.g., `2026-05-19-invoice-bulk-export.md`).
- Requirements are written in English.
- One requirement per file.
