# Curation: keeping the systems library distinctive

This is a ONE-TIME editorial pass, run by a person, over the whole `systems/` library.
It is not part of using the library and it is not part of adding one system.

- Using an existing system: go to `SKILL.md`. Nothing here applies.
- Adding one system: go to `SKILL.md`, section "Adding a design system" — that section is
  self-sufficient and is the authority. If `AUTHORING.md` exists in this folder, it is the
  fuller methodology and supersedes nothing here; if it does not exist, follow `SKILL.md`
  and do not wait for it.

Whichever of those you follow, that is a gate on ONE entry on the way in. This document is
a sweep over everything already in. They are different acts and neither replaces the other.

Run this pass once when the library is first populated, and after that only when a person
decides to — after a bulk import, or when routing starts feeling arbitrary. There is no
schedule and no trigger. Never run it per task. While `systems/` is empty there is nothing
to curate; that is the expected state until entries are added, not a problem to fix.

## Why curation, and not a better picker

A library of near-identical entries cannot be rescued by smarter selection. If three entries
distil to the same tokens and the same generic exemplars, every selection strategy over them
returns the same generic UI, and the routing decision that picked one over the others was
noise. `SKILL.md`'s scoring is deliberate arithmetic over tags — it will faithfully return a
winner from a field of indistinguishable entries, and that winner means nothing. The corpus
is the lever; the picker is not.

Two costs, both recurring, both paid forever until an editorial pass removes them:

1. Every entry holds a manifest line loaded on every routing decision.
2. Generic and boilerplate exemplars pull generated output toward exactly the default look
   this library exists to escape. They are worse than absent — they are anti-information,
   in the same sense as noise in a log you are reasoning over.

A curation pass costs a person an afternoon, once. Its savings recur on every task.

## What this pass is NOT

State this plainly because the failure mode is to "automate" it:

- No embedding model, no vector index, no similarity search, no dedup service.
- No script that runs on ingest, no hook, no scheduled job.
- No model call of any kind on the task path — before, during, or after.

Nothing in this document adds a single tool call or token to a design task. If an
implementation of this process would run at task time, that implementation is wrong. The
comparison below is done by a human reading entries side by side.

## The pass

### Step 1 — Lay the corpus out flat

List every entry: its `name`, its `aesthetic`/`project_type`/`framework` tags from
`manifest.json`, and its `spec.md` one-line purpose. Work from this list, not from the
folders. Duplicates are visible in a flat list and invisible while walking directories one
at a time.

### Step 2 — Group by pattern, not by name

Cluster entries that would produce substantially the same UI. Judge by what an implementer
would actually build from the entry:

- Token sets that differ only in a hue or one spacing step.
- Exemplar components whose markup and class conventions are interchangeable.
- Entries whose `spec.md` purpose statements are mutually substitutable.

Different names, different upstream vendors, and different file sizes are not evidence of
difference. Two entries can come from unrelated sources and still be one pattern.

This step is judgment. It has no mechanical answer and no expected output — do not try to
reduce it to a rule, and do not hand it to a model.

### Step 3 — Keep one exemplar per pattern

For each cluster, keep exactly one and drop the rest. Choose on, in strict order:

1. **Distinctiveness** — the one with a real point of view. An entry that could be any
   framework's starter theme loses to one with a committed voice, even if the generic one
   is more "complete."
2. **Quality of the extract** — complete tokens, real exemplar source (not paraphrase), a
   `spec.md` that says where NOT to use it.
3. **Provenance** — versioned and recently distilled beats unversioned or stale. A
   `source_version` of `latest`/`main`/blank counts as unversioned regardless of how recent
   `last_verified` is.

Completeness is deliberately not on this list. A large, thorough, generic entry is the exact
thing this pass exists to remove.

When two entries in a cluster are genuinely both wanted, that means they are not one pattern
— sharpen both `spec.md` files to say what distinguishes them and when each applies, then
keep both. Do not keep both while leaving the distinction implicit; that just relocates the
coin-flip into the routing decision.

### Step 4 — Drop generic entries outright, cluster or not

Independently of duplication, remove any entry that is a boilerplate scaffold or a default
starter theme with no identity, and any entry too thin to build from. These do not need a
duplicate to justify removal — they fail on their own.

### Step 5 — Log what was dropped and why

Append to `CURATION-LOG.md` in this folder, one line per removal. The file will not exist
before the first pass; create it then. It is not shipped with this skill and its absence is
not an error.

`<date> DROPPED <name> — <kept-instead-name or "generic"> — <one clause: why>`

This log is the record, not a to-do list. It is never read at task time and never read by
the skill; it exists so a later pass can tell a deliberate removal from an entry that was
never added, and so a removal can be argued with. Keep it to one line each — a log that
grows into prose is a log nobody rereads.

### Step 6 — Reconcile the manifest and stop

Remove the manifest lines for dropped entries and delete their folders. Confirm every
remaining manifest line still points at a `systems/<folder>/` that exists, and that no
`spec.md` cross-reference names an entry that is now gone. Then re-run the worked example in
`SKILL.md` — dropping entries can change which system wins it, and its authoring checklist
requires that self-test to stay truthful. Then stop. There is no task-time step.

## Worked example (deterministic — Step 3 only)

Step 3's ordering is a fixed rule, so a fixed cluster has exactly one correct survivor. Verify
it by hand; no execution and no model call are involved.

**Given** that Step 2 has placed these three entries in ONE cluster:

| Entry | Point of view | Extract | Provenance |
|---|---|---|---|
| Atlas Base | none — a default starter theme | complete tokens, exemplar source, `spec.md` has anti-patterns | `v4.2.0`, verified 2026-07-01 |
| Ember | committed: high-contrast editorial | complete tokens, exemplar source, `spec.md` has anti-patterns | `v1.1.0`, verified 2026-06-02 |
| Ember Lite | committed: high-contrast editorial | tokens only, `spec.md` paraphrases upstream marketing | `main`, verified 2026-07-20 |

**Expected outcome:** keep **Ember**; drop Atlas Base and Ember Lite.

**Expected reasoning, criterion by criterion:** criterion 1 eliminates Atlas Base outright —
it has no point of view, and its more complete extract and better provenance never get
consulted because the order is strict, not weighted. Ember and Ember Lite both survive
criterion 1, so criterion 2 decides between them: Ember has real exemplar source and a
`spec.md` with anti-patterns; Ember Lite paraphrases. Criterion 3 is never reached — and had
it been, Ember Lite's more recent `last_verified` would still have lost, because `main` is
not a version pin.

If your reasoning reached Atlas Base by way of "it's the most complete," or reached Ember Lite
by way of "it was verified most recently," you have weighted the criteria instead of ordering
them — re-read Step 3 rather than proceeding.

## Checklist

- [ ] Every entry appeared in the flat list before any judgment was made.
- [ ] Clusters were judged on produced-UI equivalence, not on names or vendors.
- [ ] Exactly one exemplar kept per cluster, chosen by the strict order, distinctiveness first.
- [ ] Any deliberately-kept pair has the distinction written into both `spec.md` files.
- [ ] Generic/boilerplate/too-thin entries removed regardless of duplication.
- [ ] Every removal has a one-line entry in `CURATION-LOG.md`.
- [ ] Manifest reconciled: no dead lines, no dangling folders, no broken cross-references.
- [ ] `SKILL.md`'s worked example re-verified against the reduced manifest.
- [ ] Nothing in this pass was wired to run automatically or at task time.
