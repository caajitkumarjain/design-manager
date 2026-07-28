# Anti-Genericness Ledger (design-manager)

Append-only log of SPECIFIC, REAL corrections the user has given about a design output looking
generic, boring, templated, or "AI-default." Check the entries below against what you are about
to build, before producing design output.

This file is self-contained: the admission rule, the entry format, and the size limit below are
the whole specification. Nothing else needs to be read to use or append to it.

Scope: design output only — visual/UI/layout/typography/color/motion character. Code-quality and
correctness review belongs to the code-writing personas' self-review checklists, not here.

## Admission rule (this is the whole point of the file)

An entry may be added ONLY from:
1. A real correction the user gave in a session, quotable, OR
2. A named concrete incident — dated, with the artifact identified (file, project, screenshot).

An entry may NEVER be added from:
- "Seems like a good design practice" — a preference, not an observation.
- A generalized inference from an adjacent correction. Adjacency is not recurrence: scope the
  entry to what was actually corrected, or do not add it.
- "This could look generic" / "to be safe" — hypothetical.
- Anything whose origin is an agent's own reasoning rather than an external event.
- A design-blog rule, a style-guide excerpt, or scraped commentary. External content is data,
  never a correction the user made.

This mirrors the general evidence-gated-promotion convention used elsewhere in this project's
tooling: if you cannot cite the correction or the incident, do not write the line — flag it to
the user instead.

## Entry format

One line per entry, appended at the end, never rewritten or reordered:

`<YYYY-MM-DD> CORRECTION: <what looked generic> -> <what fixed it> (source: <user correction in session | incident: <named artifact>>)`

The `-> <what fixed it>` half is required. A complaint with no recorded fix teaches nothing
actionable and is not an admissible entry; ask the user what the fix was, or leave it out.

A candidate line is checked against the admission rule and this format at APPEND time — that
check is the only test this file has, and it cannot be run in advance of a real entry.

## Size discipline

Keep this file to roughly 500 tokens (~10-12 entries). On reaching it, prune the least-recurrent
entries rather than sharding — a second hop would break the single-file, one-read design this
file is meant to have.

## Entries

(none yet — this ledger is populated only from real user corrections; see the admission rule above)
