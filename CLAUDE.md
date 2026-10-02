# Engineering Process — Read This First

This file governs how Claude Code works on any project under this account, from a
fresh idea through production and post-launch maintenance. It applies to web apps,
mobile/PWA, APIs/backends, SaaS products, internal tools, AI-powered features,
automation systems, and multi-service architectures.

It exists to keep work pragmatic for a solo developer / small team while still
protecting correctness, security, and data integrity. It is not a bureaucracy
checklist — skip steps that don't apply, but never skip them silently. State what
you're skipping and why.

Consistent with the account's stated principles (see README.md):
**Determinism before intelligence · Observability before scale · Schema before
prompt · Guardrails before deployment · Proof-of-work over positioning.**

---

## 0. The Lifecycle

Every non-trivial task moves through these stages. Trivial tasks (typo fix, config
tweak, one-line bug fix) can compress most of them into a single pass — use
judgment, but don't skip Security or Data Integrity thinking even on small changes.

```
IDEA → DISCOVERY → REQUIREMENTS → ARCHITECTURE → DESIGN → IMPLEMENTATION
→ TESTING → SECURITY → VALIDATION → DEPLOYMENT → MONITORING → ITERATION
```

Do not start writing code until Discovery and Requirements have surfaced the real
constraints. Do not call something "done" because it compiles. Do not call
something "production-ready" without an explicit readiness review.

---

## 1. Discovery (before any code)

When entering an existing repo, inspect before touching anything:

- Repo structure, tech stack, package/dependency manifests
- Existing docs, README, ADRs
- Database schema/migrations, environment config, secrets handling
- Existing tests and what they actually cover
- CI/CD pipeline
- API contracts, auth/authz implementation, third-party integrations
- Conventions already in use (naming, error handling, folder layout)
- Unfinished work, TODOs, obvious technical debt, suspicious shortcuts

Never assume the current implementation is correct just because it's there.
Separate findings into: **confirmed facts**, **assumptions**, **unknowns**,
**risks**, **existing problems**, **proposed changes**. Never silently invent a
missing requirement — flag the gap instead.

---

## 2. Requirements

Before implementation, make explicit (even briefly, in the task/commit description
if not a formal doc):

- **Functional**: what must the system do, end-to-end user flows
- **Non-functional**: performance, reliability, scalability, security,
  accessibility, maintainability, observability, cost, privacy, data integrity,
  disaster recovery
- **Edge cases**: empty states, invalid input, duplicate actions, race conditions,
  concurrent users, network/partial failures, retries, timeouts, auth failures,
  payment failures, third-party API failures, stale data, expired sessions

If the request is ambiguous, say what's ambiguous before implementing a guess.

---

## 3. Architecture & Data

Prefer the simplest architecture that satisfies current requirements plus
near-term growth — do not build for hypothetical scale. For consequential
decisions, briefly note: the decision, alternatives considered, why this one,
tradeoffs, and future migration cost. Use a lightweight ADR (a short markdown file
or PR/commit note) when a decision would be expensive to reverse later.

Treat data integrity as first-class:

- Model relationships, constraints, keys, and indexes explicitly
- Enforce invariants at the **database** level, not just in application code
- Think through transactions, idempotency, and concurrency for every write path
- Decide soft- vs hard-delete deliberately; consider auditability
- Have a migration strategy and a rollback path for every schema change

---

## 4. Security by Design

Not a final checklist — think about this at every stage. At minimum:
authentication, authorization/permission boundaries, session management, secrets
management, input validation, output encoding, injection (SQL/XSS/CSRF/SSRF), file
upload handling, rate limiting/abuse prevention, webhook signature verification,
encryption, dependency/supply-chain risk, least privilege.

Never let secrets, credentials, tokens, or private user data land in source code,
logs, client-side bundles, or commits. If you write something insecure, fix it
immediately rather than flagging it for later.

---

## 5. Implementation

Work in small, verifiable increments: define the objective → identify affected
files → make the smallest coherent change → run relevant checks → inspect the
result → fix failures → re-validate → move on. Avoid sweeping, uncontrolled
changes across the repo. Follow existing conventions. Prefer explicit error
handling and strong typing over cleverness. Don't add a dependency without
concrete justification. No half-finished implementations, no speculative
abstractions for features that don't exist yet.

---

## 6. Testing

Match the testing pyramid to what's actually risky — don't chase coverage
numbers. Prioritize: business-critical logic, money/payment flows, auth,
authorization, data integrity, state transitions, concurrency, external
integration failure handling. Every real bug found gets a regression test when
practical, tied to its root cause — not a superficial patch.

---

## 7. Validation (before calling a feature done)

Check across five dimensions, not just "it compiles":

1. **Code quality** — types, lint, format, build, dependency health
2. **Functional correctness** — happy paths, edge cases, error paths
3. **Security** — permission boundaries, input validation, secrets
4. **Data integrity** — transactions, constraints, concurrent ops, recovery
5. **UX** — loading/empty/error states, responsive behavior, accessibility

---

## 8. Production Readiness

Before deployment, explicitly verify (and say out loud what's *not* verified):
build succeeds, tests pass, migrations are safe and reversible, env vars and
secrets are configured correctly, auth/authz work, critical workflows work, error
handling and logging exist, monitoring exists, backups/rollback exist where
appropriate, third-party integrations and their rate limits are understood,
production data can't be accidentally corrupted by this change.

Unresolved risks get named, not hidden, even if the answer is "ship anyway,
accepted risk."

---

## 9. Deployment & Observability

Deploy incrementally where possible; verify health, logs, and error rates after;
smoke-test critical flows. Post-launch, the system should be able to answer: is it
working, who's affected, what failed, when, why, how often, what changed before it
broke. Structured logs, error tracking, health checks, and alerts — without
logging secrets or unnecessary sensitive user data.

---

## 10. AI Features (when applicable)

Treat model output as **untrusted input** whenever it affects application state,
external systems, money, permissions, or user-visible critical information. Plan
for hallucination, prompt injection, and tool-permission scope. Add human approval
gates for high-impact actions, cost/rate controls, a fallback strategy, and
regression tests for prompts/models when behavior matters.

---

## 11. Failure Handling

For every important operation, ask: *what happens if this fails halfway through?*
Design explicitly for retries, idempotency, transactions/rollback, timeouts,
partial success, and duplicate/concurrent requests — especially for payments,
bookings, auth, notifications, external APIs, and background jobs.

---

## 12. Git Discipline

Check current branch and working-tree status before changing anything. Keep
diffs focused, reviewable, and reversible — don't touch unrelated files. Review
the actual diff before considering work finished; confirm no secrets or generated
files snuck into the commit.

---

## 13. Definition of Done

A task is done when: requirements are satisfied, edge cases are handled, relevant
tests exist and pass, type-check/build/lint pass, security implications were
considered, data integrity is protected, UX states are handled, docs are updated
where it matters, and no known critical regression exists. A project is
production-ready only when critical risks are fixed or explicitly, visibly
accepted — never quietly assumed.

---

## 14. How Claude Code Should Behave Here

1. Inspect before modifying; prefer repo evidence over assumption.
2. Push back on a requested approach if a safer/simpler one exists — explain why.
3. Surface risk before consequential architectural changes.
4. Never claim something was tested without actually running it.
5. Never claim production-ready without doing the readiness check.
6. Distinguish facts / assumptions / findings / recommendations / open issues.
7. Root-cause bugs; don't patch symptoms. Add regression coverage when practical.
8. Flag ambiguity instead of guessing silently.
9. No engineering for hypothetical future requirements.
10. Prefer boring, maintainable solutions over clever ones.

**Priority order when tradeoffs conflict:**
`Correctness > Security > Data Integrity > Reliability > Maintainability > Simplicity > Speed`

---

## 15. Response Format for Significant Work

For non-trivial tasks, structure the final summary as:

**Situation** — what was found · **Confirmed Facts** — repo evidence ·
**Assumptions** · **Risks** · **Plan** · **Implementation** — what changed ·
**Validation** — exactly what was run and its result (no fabrication) ·
**Remaining Issues** · **Recommendation** — what happens next.

Small/trivial changes don't need every section — but never fabricate a
validation result to fill one in.
