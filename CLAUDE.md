# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

**Brand Image Prompt Studio** turns a blog post's keyword + markdown outline into a *Prompt Pack*:
one on-brand image prompt per image slot, with reference placeholders, generation order, alt text
and filenames. MVP generates prompts only; it does not call an image API.

As of 2026-11-08 the repo is **spec-only — no application code, package.json, or test runner exists
yet**. Do not invent build/lint/test commands; add them here once the Next.js skeleton (slice L0)
is created. Not a git repository yet.

Key files:
- `docs/Brand Image Prompt Studio — Mô tả tính năng & Đặc tả build.md` — the full product/build spec
  (features C1–C9, A1–A9, data model, RLS, Prompt Engine, roadmap L0–L4). Source of truth for scope.
- `docs/brand.json` — brand profile of the first real project (Print Your Wear Magazine). It is
  **seed/fixture data for the Brandkit tables**, never something to hard-code into app code. Its
  `visualIdentity` (colors, fonts, logo) is `unknown` and must come from Shopify theme settings.
- `.specify/memory/constitution.md` — 10 core principles + product constraints A–E (v2.0.1). Read it
  before planning or implementing; plans must pass its "Constitution Check". Notable: no push, merge,
  deploy or remote migration without Hung's explicit approval each time; ADRs go in `docs/adr/`.
- `specs/001-prompt-pack-mvp/spec.md` — approved spec for slices L0+L1 (with a Clarifications section that
  overrides the source doc where they differ).
- `docs/style-guides/` — one Markdown style guide per blog category per project (frontmatter = machine
  params, body = guidance fed to AI). Print Your Wear skeletons live in `print-your-wear/`.

## Workflow (Spec Kit)

Features go through the Spec Kit skills in `.claude/skills/`:
`/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`
(`/speckit-analyze`, `/speckit-checklist`, `/speckit-converge` as needed). Helper scripts are
PowerShell (`.specify/scripts/powershell/*.ps1`); feature numbering is sequential.

Build in thin slices and don't start a slice before the previous one passes acceptance:
L0 foundation (migrations + RLS + invite auth + seed purpose templates) → L1 Creator flow (C1–C6,
engine A–F, provider via env) → L2 Admin UI → L3 quality/feedback/logs → L4 in-app image gen (deferred).

## Planned architecture (from spec §7–9)

Stack: Next.js App Router + TypeScript on Vercel, Supabase (Auth, Postgres with RLS, private
Storage), `openai` Node SDK with custom `baseURL`, Zod.

**Prompt Engine pipeline — the core idea.** AI runs in two steps and writes the descriptive part of
each prompt; code owns everything mandatory (spec 001 clarifications override source doc §7.5):

| Step | Who | Where (planned) |
| --- | --- | --- |
| A Context Resolver — snapshot brandkit, category style guide, style sets, purpose templates into `sessions.context_snapshot` (once, at first analysis) | code | `lib/engine/resolver.ts` |
| B Slot Planner — outline → image slots JSON | AI call 1 | `app/api/sessions/[sid]/plan` |
| C Creator edits slots | human | creator UI |
| D Prompt Writer — descriptive prompt body + alt text + filename for all images (batch of 10 if > 20) | AI call 2 | `app/api/sessions/[sid]/prompts` |
| E Prompt Finisher — append "dont", negative, reference instructions + placeholders, aspect, brand colors only if a palette exists | code | `lib/engine/assembler.ts` |
| F Validator — placeholders ↔ references, required refs, GEN cycles, length, banned words, brand HEX (conditional); also re-run on manual edits | code | `lib/engine/validator.ts`, `graph.ts` |

Cross-cutting rules that span multiple files:
- Context precedence: system defaults → Brandkit → Category (style guide) → Style Set → Purpose
  template → Creator options; Brandkit "dont"/banned words always win.
- L1 stores no image files: references are names/descriptions only. No per-user AI call limit; only a
  provider-wide daily limit.
- Placeholders use `[[TYPE:KEY]]` (`REF:LOGO`, `REF:PRODUCT:<key>`, `REF:MODEL:<key>`,
  `REF:DESIGN#n`, `GEN:S1-1`, `TEXT:<name>`); every placeholder has exactly one `references` entry.
  `gen_order` is a topological sort of `[[GEN:…]]` deps.
- AI output: JSON Schema structured output, re-validated with Zod; schemas defined once in
  `lib/engine/schemas.ts`. Non-JSON-schema providers: JSON mode + Zod + one retry.
- Providers implement the `AIProvider.generateJSON` interface (`lib/ai/provider.ts`); the
  `ai_providers` table stores only the env var *name* of the key.
- Presets (brandkit, style_sets, purpose_templates) are versioned and never overwritten once
  published; prompts record `model` and `template_versions`. Bump the version of the B/D system
  prompts whenever they change.
- Authorization is RLS (`is_admin()`, `is_member(project_id)`); every migration touching a
  `project_id` table ships its policies. `activity_logs` is written only server-side via service role.
- `lib/engine` should stay pure (no DB/AI calls) so it is unit-testable with a fake provider.

## Conventions

- UI text is Vietnamese; generated prompts, alt text and filenames are English.
- Spec docs are written in Vietnamese; keep new specs/plans in Vietnamese to match.
