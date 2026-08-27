# Agent instructions — QVL QA Knowledge Base

This repo is a QA knowledge base for **Qikify Volume Discounts** (Testlane
project key `QVL`), meant to be read and edited by AI agents (Claude Code,
ChatGPT, etc.) as well as humans. If you are an agent working in this repo,
follow these conventions.

## What this repo is for

Capture business rules, edge cases, decisions, and known issues at the level
of detail needed to write and maintain QA test cases — not a merchant-facing
help doc. See [README.md](README.md) for the human-facing overview.

## Where things go

- `product/overview.md` — what the app does, at a glance, for someone new to it.
- `product/glossary.md` — terms used inconsistently elsewhere (tier, break, bundle, combo...). Add a term here the first time you find it used ambiguously.
- `product/feature-map.md` — how features interact/conflict/stack. Update this whenever a change affects more than one feature.
- `features/<slug>.md` — one file per feature. Purpose, business rules, edge cases. **Do not add a Known Issues section here** — known issues live centrally in `known-issues/known-issues.md`, tagged by feature, so they don't drift out of sync with two copies.
- `testing/regression-checklist.md` — what to re-check after any change, across features.
- `testing/test-data.md` — reusable test data/store setup needed to exercise these features.
- `testing/environments.md` — staging URLs, accounts, how to get access.
- `decisions/decision-log.md` — append-only log of "why is it this way" — one entry per decision, dated, with the reasoning. Never delete entries, only add.
- `known-issues/known-issues.md` — current known bugs/limitations, tagged by feature slug so they're easy to filter.
- `dev-references/references.md` — links out: Jira component, related repos, API docs, Slack channels.

## Status convention

Every `features/*.md` file starts with `**Status:** skeleton|in-review|verified`
at the top. Bump it as you verify content against real tickets/current app
behavior. Don't mark `verified` from the help center alone — it's known to lag.

## Workflow for filling in a feature

1. Pull related Jira tickets / changelog entries for the feature.
2. Use the `/analysis` skill (Claude Code) against those tickets to extract
   current business rules, edge cases, gaps.
3. Update the feature file's Purpose / Business Rules / Edge Cases, bump status.
4. If the change affects other features, update `product/feature-map.md`.
5. If it's a "why is it built this way" decision, add an entry to
   `decisions/decision-log.md` instead of burying the reasoning in the feature file.
6. Re-upload changed files into Testlane's QVL project context (it does not
   sync live from git — see README.md).

## Editing conventions

- Keep feature files scoped to one feature. Cross-feature behavior goes in `feature-map.md`.
- Prefer editing existing files over creating new ones — only add a new file under `features/` for a genuinely new feature, matching the existing template.
- Cite the source of a claim (Jira ticket ID, help center URL, or "verified by direct testing on staging") — don't state a business rule without saying where it came from.
