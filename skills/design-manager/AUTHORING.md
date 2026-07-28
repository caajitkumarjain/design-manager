# Authoring: adding a design system to the library

This document ELABORATES the "Adding a design system" section of `SKILL.md` in this same
folder. It adds no new rule and overrides nothing: `SKILL.md` states the four-point
distillation methodology inline and works standalone, so if this file is ever deleted the
authoring path still functions exactly as `SKILL.md` describes. Read this only when you
want the fuller procedure around those four points.

It is not needed to USE a system. If you are building UI from a system already in the
manifest, stop reading and go back to `SKILL.md`.

Where this document and `SKILL.md` ever appear to disagree, **`SKILL.md` wins** — it is the
skill; this is commentary on it.

Run this process once per system. The entire point is that the compression happens here, at
authoring time, so no future task ever loads the raw source material.

## Why once, and why compressed

A design system's upstream source — a component-library repo, a docs site, a tutorial, a
set of reference screenshots — is large, mostly redundant, and mostly irrelevant to
generating UI. Storing it raw would mean every future task touching this system pays for
the whole dump. Distilling once means every future task pays only for the compact entry.

The consequence to hold onto is the same one `SKILL.md` states as its third distillation
point — entry size is governed by the caps, never by the source's size. Its falsifiable
form: **if a distilled entry's size tracks its source's size, the distillation did not
happen.** Go back to step 2.

## Step 1 — Gather the raw source

Collect whatever you have: the upstream repo or its docs, the published token file, the
CSS/Tailwind config, tutorial pages, screenshots. Keep it OUTSIDE `systems/` — a scratch
directory, a browser tab, a repo cloned elsewhere. Nothing from this step is stored; this
is `SKILL.md`'s fourth point ("read the upstream source once, at authoring time, and never
store it") as a concrete instruction about where to put things meanwhile.

Record two facts now, for the manifest entry in step 4: the canonical source URL, and the
exact release tag or commit hash you are reading. Capture the pin at the moment you read,
not afterwards from memory — reconstructing "which version was that" later is the failure
this step exists to prevent.

## Step 2 — Extract only the structured essentials

`SKILL.md` defines what goes in each file. This step is about what to leave behind.

Take these and nothing else:

1. **Design token values** — the color ramp, type scale (family, sizes, weights,
   line-heights), spacing scale, radii, shadows, motion durations and easings. Verbatim
   numbers in the system's own naming. Highest-value extract, usually the smallest.
2. **The usage rules the system actually states** — the two or three conventions it would
   be wrong to violate, in the system's own terms.
3. **At least one anti-pattern** — what this system deliberately does NOT do, and where it
   should not be used. `SKILL.md` treats a spec without one as under-distilled, and it is
   the part that stops a future session sliding back to generic defaults, so extract it
   deliberately rather than hoping it falls out of the prose.

Explicitly do NOT extract: the full component catalogue, build tooling, tests, changelogs,
marketing copy, tutorial narrative prose, or any component a competent implementer would
derive from the tokens and the stated conventions.

If a system publishes verbatim CSS or theme variables a consuming project would paste in,
those belong in the optional `css/` directory — a destination for file copies, never
context to read. Nothing in `css/` counts toward the distillation; if you find yourself
reading from it to write `spec.md`, you are still in step 2.

If two candidate systems distil to near-identical tokens and rules, add one and note the
other as an alias in its `spec.md`. Near-duplicate entries are anti-information: they add
index weight and hand routing a coin-flip it cannot win.

## Step 3 — Write the entry

Create `systems/<folder>/` and write `tokens.json` and `spec.md` per the folder convention
and the caps in `SKILL.md`. Do not invent a variant layout — selection depends on the shape
being uniform, and `<folder>` must match the manifest's `folder` field exactly.

If it will not fit the caps, cut usage prose before cutting token values. Values are what
`tokens.json` is authoritative for; prose is recoverable, a dropped scale step is not.

## Step 4 — Record provenance and index it

Provenance lives in the MANIFEST entry, not in `spec.md`. Fill the three fields the
Manifest schema section of `SKILL.md` requires — `source_url`, `source_version`, and
`last_verified` — using the URL and pin captured in step 1. `source_version` must be a real
tag or commit hash: never `"latest"`, `"main"`, or blank. A verification date without a pin
records that a comparison happened but not against what, so the two only work as a pair.

Then append the manifest entry with its routing tags, drawn only from the vocabularies in
`SKILL.md`'s Selection step 2 — a tag outside those vocabularies can never be matched and
is dead weight. Keep the entry compact: the manifest is the only part of this work that is
read on every routing decision.

## Step 5 — Finish per SKILL.md, then stop

Adding an entry is a material edit to the skill, so complete the closing steps `SKILL.md`
already specifies: re-check the token budget and shard the manifest if it crossed the
threshold; re-run the Worked example against the updated manifest and update its
expected-output table in the same edit if the new entry changes the winner (a stale
self-test is worse than none); and run a skill-review pass before shipping.

There is no task-time step. Do not return to this document during ordinary use. If upstream
moves, that is a fresh authoring pass: repeat steps 1-4 for that system and update
`source_version` and `last_verified`.

## Checklist

- [ ] Nothing from the raw source was copied into `systems/<folder>/` verbatim in bulk.
- [ ] `tokens.json` and `spec.md` present, named and capped per `SKILL.md`.
- [ ] `tokens.json` is values only — numbers verbatim, nothing rounded, averaged, or invented.
- [ ] `spec.md` states what the system is for AND at least one anti-pattern.
- [ ] `css/` (if used) holds copied files only; nothing was read from it to write the entry.
- [ ] Entry size is unrelated to source size.
- [ ] `<folder>` matches the manifest `folder` field exactly.
- [ ] Manifest entry added: routing tags from the listed vocabularies only, plus
      `source_url`, `source_version` (a real pin), and `last_verified`.
- [ ] No near-duplicate of an existing entry, or the duplicate is recorded as an alias.
- [ ] Token budget re-checked, Worked example re-run, review pass completed.
