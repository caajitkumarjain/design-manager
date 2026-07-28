# Provenance Schema (design-manager)

Scope: the `provenance` object on each `manifest.json` entry. This doc ELABORATES the
schema stated in `SKILL.md`; it never replaces it. If the two ever disagree, `SKILL.md`
wins and this file is the stale one. Deleting this file is always safe — `SKILL.md`'s
three-field minimum stands on its own by its own declaration.

Provenance lives on the manifest ENTRY, not in `spec.md`. `spec.md` carries usage prose and
anti-patterns only.

## Required fields (restated from SKILL.md — authoritative there)

- `source_url` — canonical upstream page or repo the system was distilled from. One URL.
  Not a search result, not a mirror, not an aggregator.
- `source_version` — the exact upstream release tag or commit hash distilled. Never
  `"latest"`, `"main"`, or blank.
- `last_verified` — ISO date (`YYYY-MM-DD`) the entry was last compared to upstream at the
  pinned `source_version`. A verification date, not a file-edit date.

### Why the pin and the date only work as a pair

Without `source_version`, `last_verified` says a comparison happened but not against WHAT —
the entry cannot be re-checked, only re-guessed. The pin is what makes the date falsifiable:
any curator can return to that exact upstream state and confirm or refute the entry.

## Optional field

- `curator_notes` — one or two lines: what was deliberately dropped in distillation, known
  divergences from upstream, or why this system is stored at all. Omit it entirely rather
  than writing an empty string. Its absence is never a schema failure.

## When these are written

Once, at authoring/curation time, per `SKILL.md`'s "Adding a design system" step 2 — and
again only when a human re-curates the entry. Never written, touched, or recomputed by a
design task that merely reads the entry.

## When these are read

Never by selection. `SKILL.md` states plainly that selection does not read `provenance`;
only staleness audits do. A curator reads these fields deliberately, on a curation pass.

## What is explicitly NOT done

- No runtime freshness check. Nothing fetches `source_url` to see if upstream moved.
- No HTTP request, no network access, no upstream diff — per task, per session, or ever.
- No background job, no scheduled revalidation, no auto-expiry of stale entries.
- No agent may edit these fields to "refresh" an entry it did not actually re-verify against
  upstream at the pinned version. Bumping `last_verified` without doing the comparison
  converts the field from evidence into decoration, silently.

Staleness surfaces only when a human re-curates and reads the dates. An old `last_verified`
is information, not an error, and not a trigger for any automated action.
