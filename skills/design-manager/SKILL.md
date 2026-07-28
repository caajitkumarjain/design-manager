---
name: design-manager
description: >
  Selects ONE curated design system from a local indexed corpus and loads only that
  system's tokens and spec, so UI work is grounded in a real system instead of generic
  defaults. Deterministic tag lookup against a local manifest — no LLM call, no embedding
  call, no network access, and never loads more than one system per task. Side effects:
  READ-ONLY on this skill's own folder; writes nothing. Triggers on: "design system",
  "style this", "make it look like", "pick a design system", "UI kit", "design tokens",
  "brand palette", "component library styling", or any request to build/restyle a UI where
  no design system has already been chosen in this session.
tools: Read, Glob
---

# design-manager

Ground UI work in ONE real, curated design system. Never load the corpus.

## Not for

- Chart/graph/dashboard color and mark decisions — use a dedicated data-visualization skill.
- Published Artifact/page-level design systems — use your platform's own design-guidance skill.
- Sourcing a system that isn't already indexed here. Adding one is an authoring task
  (see "Adding a design system"), never something to do mid-task.

## Selection (deterministic — no model judgment, no extra call)

1. Read `manifest.json` in this skill's folder. If it has been sharded (see Scaling),
   read only the shard whose letter range covers your candidate names; if you have no
   candidate name yet, read every shard — they are jointly capped at the same budget.
2. Extract the request's **selection facts** from what the user actually wrote, in this
   priority order. Do not infer facts the user did not state.
   - `system` — a system named outright ("use Material 3"). An explicit name wins
     immediately; skip to step 4.
   - `framework` — react, vue, svelte, next, tailwind, plain-css, react-native.
   - `aesthetic` — the user's own adjective: minimal, brutalist, editorial, corporate,
     playful, dense, glassy, retro.
   - `project_type` — dashboard, marketing-site, docs, mobile-app, admin-panel,
     data-tool, landing-page.
3. Score every manifest entry: **+3** per matched `aesthetic` tag, **+2** per matched
   `project_type` tag, **+1** per matched `framework` tag. Case-insensitive exact string
   equality on tags — no fuzzy matching, no synonym expansion, no semantic similarity.
   Highest total wins. Ties break by lower `rank` field, then alphabetically by `name`.
   This is arithmetic; do not deliberate over it.
4. Load ONLY the winner: read `systems/<folder>/spec.md` and `systems/<folder>/tokens.json`.
   Nothing else. Do not read a second system "to compare" — comparison is the exact cost
   this index exists to avoid.
5. State the choice in one line before writing any UI code: `design-manager: <name>
   (score <n>) — <the tags that matched>`. If the score is ≤2, say so; a weak match is
   worth the user knowing about.

## Worked example (deterministic self-test)

The selection arithmetic above has exactly one correct answer for any fixed manifest and
request. Use this to verify the mechanism by hand — no LLM call, no execution needed.

**Given this manifest:**

```json
[
  { "name": "Alpha UI", "folder": "alpha-ui",
    "aesthetic": ["minimal", "corporate"], "project_type": ["dashboard", "admin-panel"],
    "framework": ["react", "vue"], "rank": 1 },
  { "name": "Brutal Kit", "folder": "brutal-kit",
    "aesthetic": ["brutalist"], "project_type": ["marketing-site", "landing-page"],
    "framework": ["plain-css"], "rank": 2 },
  { "name": "Console", "folder": "console",
    "aesthetic": ["minimal", "dense"], "project_type": ["dashboard", "data-tool"],
    "framework": ["react", "svelte"], "rank": 3 }
]
```

**Given this request:** "Build me a minimal, dense React dashboard for our internal metrics."

**Extracted selection facts:** `aesthetic` = minimal, dense; `project_type` = dashboard;
`framework` = react. No system named outright.

**Expected scoring:**

| Entry | aesthetic (+3 ea) | project_type (+2 ea) | framework (+1 ea) | Total |
|---|---|---|---|---|
| Alpha UI | minimal → 3 | dashboard → 2 | react → 1 | **6** |
| Brutal Kit | none → 0 | none → 0 | none → 0 | **0** |
| Console | minimal, dense → 6 | dashboard → 2 | react → 1 | **9** |

**Expected output:** `design-manager: Console (score 9) — minimal, dense, dashboard, react`
and only `systems/console/spec.md` + `systems/console/tokens.json` are read.

If your arithmetic disagrees with this table, you have applied a weight, a match rule, or a
tiebreak incorrectly — re-read Selection step 3 rather than proceeding. Note that `rank`
never entered this example: it is a tiebreak only, and Console wins outright on score
despite holding the worst rank of the three.

## No match

If nothing scores above 0 — including when `systems/` is empty, which is its state until
systems are added — say exactly that: no indexed design system matches, and you are
proceeding without one. Then do the UI work normally. Never invent a system name, never
describe an unindexed system from memory, and never present ungrounded output as if it
came from the corpus. A silent miss is the one failure mode that makes this skill worse
than not having it.

## Using the loaded system

`tokens.json` is authoritative for color, type scale, spacing, radius, shadow, and motion
— use its values verbatim rather than approximating them. `spec.md` carries the
component conventions and the things this system deliberately does NOT do. Where the two
disagree, `tokens.json` wins for values and `spec.md` wins for usage. Where the user's
explicit instruction disagrees with either, the user wins — say which token you departed
from.

## Folder convention

```
skills/design-manager/
  SKILL.md
  manifest.json
  systems/
    <system-name>/          kebab-case, matches the manifest `folder` field exactly
      tokens.json           values only — colors, type, spacing, radius, shadow, motion
      spec.md               usage prose + anti-patterns; ≤2,000 chars (≤500 tokens)
      css/                  optional; verbatim upstream CSS/vars, never read wholesale
```

`tokens.json` is capped at ≤2,000 chars. `css/` exists for verbatim upstream artifacts a
project may copy from; it is a destination for file copies, not context to read.

## Manifest schema

Array of entries. Every field required; unknown fields are ignored by selection.

- `name` — display name, e.g. `"Material 3"`.
- `folder` — kebab-case directory under `systems/`, e.g. `"material-3"`.
- `aesthetic` — array of aesthetic tags (weight 3).
- `project_type` — array of project-type tags (weight 2).
- `framework` — array of framework tags (weight 1).
- `rank` — integer tiebreak; lower is preferred. Reflects curation confidence.
- `provenance` — object recording where the entry came from and when it was last checked.
  Selection never reads it; staleness audits do. Minimum viable shape, sufficient on its
  own:
  - `source_url` — canonical upstream page or repo the system was distilled from.
  - `source_version` — the exact upstream release tag or commit hash distilled. Never
    `"latest"`, `"main"`, or blank: a verification date without a pin records that a
    comparison happened but not against what, so the two fields only work as a pair.
  - `last_verified` — ISO date (`YYYY-MM-DD`) the entry was last compared to upstream.

  If `PROVENANCE-SCHEMA.md` exists in this skill's folder, it is the fuller specification —
  read it and follow it. **If that file does not exist, the three fields above are the
  schema; use them as written and do not wait for anything.**

Use only tags from the vocabularies listed in Selection step 2. A tag outside those
vocabularies can never be matched and is dead weight.

## Scaling

Past ~20 entries the manifest exceeds a ~500-token reference-chunk limit. At that point
shard it into `manifest-a-f.json`, `manifest-g-m.json`, `manifest-n-s.json`,
`manifest-t-z.json` by the first letter of `name`, and list the shard filenames in this
section. Sharding changes which file is read, never the selection arithmetic.

## Adding a design system

Authoring-time work, done deliberately and never mid-task. This section is self-sufficient:
follow it as written.

1. **Distil the upstream system** into `tokens.json` + `spec.md` within the caps above.
   Distillation, not summarisation: extract the system's actual decided VALUES and its
   stated usage rules; never paraphrase marketing prose. Concretely —
   - `tokens.json` carries values only, no commentary: the color ramp, the type scale
     (family, sizes, weights, line-heights), the spacing scale, radii, shadows, and motion
     durations/easings. Copy numbers verbatim from upstream; do not round, average, or
     invent a missing step.
   - `spec.md` carries usage prose and, importantly, the anti-patterns — what this system
     deliberately does NOT do. A spec without at least one anti-pattern is under-distilled;
     the anti-patterns are what stop a future session sliding back to generic defaults.
   - The entry's size is governed by the caps, never by the source's size. A large upstream
     system does not earn a larger entry — it earns harder selection. If it will not fit,
     cut usage prose before cutting token values.
   - Read the upstream source once, at authoring time, and never store it. The raw source
     is not an artifact of this skill.

   If `AUTHORING.md` exists in this skill's folder, it is the fuller methodology — read it
   and follow it. **If that file does not exist, the four points above are the methodology;
   apply them and proceed. Do not block on a file that is not there.**
2. **Record provenance** in the manifest entry per the Manifest schema section above
   (`source_url`, `source_version`, `last_verified`).
3. **Create `systems/<folder>/`** and append one manifest entry, tags drawn only from the
   vocabularies in Selection step 2.
4. **Re-check this skill against the token budget:** SKILL.md ≤3,000 tokens, each chunk
   ≤500, manifest sharded if it crossed the threshold.
5. **Re-run the worked example** in this file against the updated manifest. Adding an entry
   can change which system wins that example; if it does, update the expected-output table
   in the same edit so the self-test stays truthful. A stale self-test is worse than none.
6. **Run a skill-review pass** before shipping the change — any material edit to a skill,
   including one that changes what it routes to, deserves a review gate rather than a
   silent commit.
