# Worked-Example Walkthrough

This walks through the deterministic selection mechanism end to end, using the fixed
example shipped in `SKILL.md`, so you can verify the arithmetic by hand before trusting
it on a real request.

## The manifest

Three curated entries, each scored against a request's tags:

| id | framework | aesthetic | project type | rank |
|---|---|---|---|---|
| `console` | react | minimal-technical | dashboard | 1 |
| `alpha-ui` | react | corporate | dashboard | 2 |
| `brutal-kit` | vue | brutalist | marketing | 3 |

## The request

> "Build a dashboard in React, minimal and technical, nothing decorative."

Tag extraction: `framework=react`, `aesthetic=minimal-technical`, `project_type=dashboard`.

## The scoring pass

Fixed point weights: framework match = 3, aesthetic match = 3, project-type match = 2,
rank tiebreak = 1 (only applied among ties, highest rank wins).

| id | framework | aesthetic | project type | total |
|---|---|---|---|---|
| `console` | 3 | 3 | 2 | **8** |
| `alpha-ui` | 3 | 0 | 2 | 5 |
| `brutal-kit` | 0 | 0 | 0 | 0 |

No tie at the top, so the rank field never enters the decision. `console` wins outright.

## What loads into context

Only `systems/console/tokens.json` and `systems/console/spec.md` are read. `alpha-ui` and
`brutal-kit`'s token files are never touched — the manifest's small metadata is the only
thing scanned across the whole library, which is what keeps this mechanism flat-cost
regardless of how large the library grows.

## Why this matters

The point of the exercise isn't the specific numbers — it's that the outcome is
reproducible by a human with a calculator, not a judgment call an LLM could render
differently on a re-run. If you extend the manifest, re-run this exact table against your
new entries before trusting the skill on a real request; `SKILL.md`'s own authoring step
requires this re-verification whenever the manifest changes.
