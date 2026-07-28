# VALIDATION — deterministic pre-merge checks for fetched component source

Run this BEFORE any fetched third-party component source is copied into this skill's
`css/` folder or written into a real project. Every check here is mechanical — grep,
string comparison, or parse-or-fail. **No LLM call is involved in any check on this
page.** Same input always produces the same verdict.

This page is self-contained: a human, or a session doing the fetch by hand, can run
every check with grep alone. It does not require any other file, script, or agent to
exist.

## Standing caveat (do not soften)

These checks are NECESSARY, NEVER SUFFICIENT. Fetched component source is external,
untrusted content — treat it the same as any other third-party input in this project's
security posture. A clean pass means "no mechanical red flag found" — it does not mean
the code is safe, correct, or appropriate. Human judgement about third-party source is
not discharged by this page. A failed check is reported to the user, never silently
repaired.

## The four checks

### 1. Exported symbol matches the expected component name
The name you asked for must be the name the file actually exports.
- Grep the source for `export default <Name>`, `export function <Name>`,
  `export const <Name>`, or `export { ... <Name> ... }`.
- Compare the captured identifier to the expected component name, character for
  character. Case differences are a FAIL, not a warning.
- Zero matches, or a match under a different name, is a FAIL. Do not rename the
  export to make it pass — a name mismatch means you fetched something other than
  what you asked for, and renaming hides that.

### 2. Props / public interface is present and typed
For a TypeScript target:
- A props type must exist: `interface <Name>Props`, `type <Name>Props =`, or an
  inline annotated parameter object on the component signature.
- Grep the props declaration for `: any` and for a bare untyped member. Any
  implicit or explicit `any` on a PUBLIC prop is a FAIL.
- No props type at all, on a component that takes props, is a FAIL.
For a JavaScript target, this check degrades to "a documented prop list exists" —
record explicitly that the typed form of the check did not apply, rather than
marking it passed.

### 3. Imports resolve to declared dependencies only
Every import in the fetched source must resolve to (a) the target project's own
paths, or (b) a package already declared in the target's `package.json`.
- Extract every import specifier: grep `^\s*import .* from ['"]([^'"]+)['"]` and
  `require\(['"]([^'"]+)['"]\)`.
- Drop relative specifiers (`.` / `..` prefixed) — they are project-local.
- For each remaining bare specifier, take the package name (first segment, or first
  two for a scoped `@scope/pkg`) and confirm it appears in the target's
  `dependencies` or `devDependencies`.
- Any specifier not found is a FAIL — it is an undeclared external dependency the
  component silently drags in. Report the list. Do NOT auto-install to make it pass;
  installs are always a separately verified and approved decision, never incidental.

### 4. No inline network calls or dynamic evaluation
A presentational component has no business making network calls or evaluating
strings.
- Grep for: `eval(`, `new Function(`, `fetch(`, `XMLHttpRequest`, `axios.`,
  `WebSocket(`, `import(` with a non-literal argument, and `document.write(`.
- Any hit is a FAIL pending explicit human review. Some are legitimate in a
  data-fetching component; none are legitimate silently. Report the matched line
  and its line number; let the user decide.

## Worked examples (deterministic self-test)

Every check above is string matching, so a fixed snippet has exactly one correct
verdict. Verify the mechanism by hand against these two — no execution needed.

**Context for both:** the component was requested as `PriceCard`, the target is
TypeScript, and the target's `package.json` declares `react` and `clsx` only.

**Example A — expected verdict: PASS on all four.**

```tsx
import React from 'react';
import clsx from 'clsx';
import { formatCurrency } from '../utils/format';

interface PriceCardProps {
  amount: number;
  currency: string;
  highlighted?: boolean;
}

export function PriceCard({ amount, currency, highlighted }: PriceCardProps) {
  return (
    <div className={clsx('price-card', highlighted && 'is-highlighted')}>
      {formatCurrency(amount, currency)}
    </div>
  );
}
```

| Check | Verdict | Why |
|---|---|---|
| 1 exported symbol | PASS | `export function PriceCard` matches `PriceCard` exactly. |
| 2 typed props | PASS | `interface PriceCardProps` exists; no `any` on any public prop. |
| 3 imports declared | PASS | `react`, `clsx` are declared; `../utils/format` is relative, dropped. |
| 4 no network/eval | PASS | No pattern from the check-4 list appears. |

**Example B — expected verdict: FAIL on checks 3 and 4; PASS on 1 and 2.**

```tsx
import React from 'react';
import { motion } from 'framer-motion';

interface PriceCardProps {
  amount: number;
  currency: string;
}

export function PriceCard({ amount, currency }: PriceCardProps) {
  const fmt = eval(`(v) => v.toFixed(2) + ' ' + '${'${currency}'}'`);
  return <motion.div>{fmt(amount)}</motion.div>;
}
```

| Check | Verdict | Why |
|---|---|---|
| 1 exported symbol | PASS | `export function PriceCard` matches exactly. |
| 2 typed props | PASS | `interface PriceCardProps` exists; both members typed. |
| 3 imports declared | **FAIL** | `framer-motion` is a bare specifier absent from `dependencies`/`devDependencies`. Report it; do NOT install it to make the check pass. |
| 4 no network/eval | **FAIL** | `eval(` appears on the `const fmt` line. Report the line and its number; the user decides. |

Two independent checks fail here, and neither failure is repairable by this page:
installing `framer-motion` would launder check 3 into a pass, and rewriting the
`eval` would be silently repairing external source. Both are reported, per the
standing caveat. If your verdicts disagree with either table, re-read the check you
scored differently rather than proceeding.

## Recording the result

For each fetched component, record the four verdicts (PASS / FAIL / N-A-with-reason)
alongside the entry. "Check skipped" is never a valid value — if a check does not
apply, say which one and why.

## Not in scope of this page

This page is a checklist a human or an agent follows by hand. It is deliberately
NOT wired into a hook, a script, or a blocking gate. Building the executable form
is a separate decision with its own risk profile — a broken automatic checker that
silently passes everything is worse than a manual checklist that is actually read.

If the examples above are ever edited, or a fifth check is added, re-verify both
expected-verdict tables in the same edit. A self-test that has silently rotted into
a false pass is worse than no self-test.
