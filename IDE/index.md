# Cursor: quick guide for new users

Short onboarding for **workflow**, **Cursor Rules**, **MCP**, and **repeatable use cases**. Treat the AI like a capable teammate with full repo context: steer it with clear goals, narrow `@` scope, and review every diff.

**Learn more:** [Cursor Docs](https://cursor.com/docs) · [Rules](https://cursor.com/docs/rules) · [MCP](https://cursor.com/docs/mcp)

---

## Quick start (2–3 minutes)

1. Install Cursor, sign in, and **open your project folder as the workspace root** (not a parent directory unless you mean to index everything under it).
2. Wait until **indexing** has settled so `@`/search ground truth matches the files you care about.
3. **Chat** (or Ask mode): best for questions, exploration, and small clarifications—less autonomous editing.
4. **Agent**: best when you want multi-step work—reading files, running commands, and applying edits across the tree. Some versions label similar flows **Composer**; same idea: _agentic editing with tools_.

---

## Core workflow

### Give the model the right context

- Use **`@`** to pull in files, folders, and (where available) docs or web. Prefer **small, precise paths** over “the whole repo.”
- Paste **stack traces**, **logs**, and **error messages** verbatim.
- Select code in the editor and reference it (**“this selection”** / **Add to chat**) when the bug or change is local.

### Editing loop

1. State the **goal** and **non-goals** (e.g. “fix bug X; do not change public API”).
2. **`@`** the smallest set of directories or files that contain the truth.
3. Let the agent propose changes; **read the diff** like a code review.
4. **Run tests or the app** from the integrated terminal; paste failures back in.
5. Repeat until done.

### Terminal and Git

- The agent can **run shell commands** when useful; **approve** anything destructive or network-facing if prompted.
- Use the **Source Control** view; **commit only after** you understand and agree with the diff.

### Privacy and sensitive paths

- Assume prompts and referenced code can be processed like a **remote collaborator** with repo access—use **`.cursorignore`** (and normal `.gitignore` patterns where applicable) for secrets, credentials, dumps, and large binaries you never want in context.

---

## Cursor Rules

Rules **persist** preferences and project law so you do not repeat the same instructions every chat.

| Kind              | Where                        | Use for                                                                    |
| ----------------- | ---------------------------- | -------------------------------------------------------------------------- |
| **User rules**    | Cursor **Settings** (global) | How you like answers, tone, habitual constraints, personal defaults.       |
| **Project rules** | `.cursor/rules/*.mdc`        | Stack conventions, architecture, APIs, security, PR style—**team shared**. |

Optional: an **`AGENTS.md`** (or similar) at the repo root can complement rules with **human-oriented** onboarding; rules remain the structured, scoped layer the agent loads reliably.

### `.mdc` shape (minimal template)

Create `.cursor/rules/` and add focused files. Example:

```markdown
---
description: Example TypeScript conventions for this repo
globs: "**/*.ts"
alwaysApply: false
---

- Prefer explicit return types on exported functions.
- Do not add new dependencies without a one-line justification in the PR.
```

**Frontmatter quick reference**

- **`description`**: Short label (shown when picking rules).
- **`globs`**: File pattern—rule applies when matching files matter to the session.
- **`alwaysApply: true`**: Injected broadly; keep these **short** and truly universal.

**Best practices:** one topic per rule; **under ~50 lines** where possible; **concrete examples** (“do / don’t”); avoid pasting **secrets** into rules.

---

## MCP (Model Context Protocol)

**What it is:** A standard way to plug **external tools and data** into the agent—browser automation, issue trackers, observability, design tools, internal APIs, and more—so the model can call **structured tools** instead of guessing.

**In Cursor:** Configure MCP servers under **Cursor Settings → MCP** (names and layout follow your Cursor version). After adding or changing a server, **restart** if needed and complete any **OAuth / API key** flows the server requires.

**Safety:** Prefer **read-only** or **scoped** credentials; confirm **tool prompts** before approve; grant the **minimum** access the workflow needs.

**When it pays off**

- **Live** data: tickets, customer issues, traces, metrics—ground answers in what is true _now_.
- **Verification**: drive a browser to reproduce a bug or confirm a UI fix.
- **Design handoff**: pull structured context from design tools instead of pasting screenshots alone.

To build or integrate a server, start from the [MCP documentation](https://cursor.com/docs/mcp) and follow the linked guides there.

---

## Use cases (copy-paste patterns)

Broader playbooks: **product/architecture brief → Cursor plan** in [implement-new-feature.md](implement-new-feature.md); **greenfield** (design-first UI + backend stack rules) in [init-new-project.md](init-new-project.md). **Systematic debug** (local, logs, CI): [debug.md](debug.md). **Writing tests** (unit/integration/regression/contract/UI): [write-test-case.md](write-test-case.md).

Each pattern below: **Goal → What to do → Tip.**

### Onboard to an unfamiliar area

- **Goal:** Understand how a subsystem works end-to-end.
- **What to do:** `@` the main package or service folder. Ask for a **data-flow summary**, entry points, and “where would I add feature X?”
- **Tip:** Ask for a **mermaid diagram** only when the flow is genuinely branching; otherwise keep it prose + file links.

### Fix a bug from a stack trace

- **Goal:** Reproduce and patch with minimal blast radius.
- **What to do:** Paste the **full** trace. `@` the file and line from the top frame; if tests exist, `@` the nearest test file.
- **Tip:** Request a **regression test** in the same change when feasible.

### Refactor behind guardrails

- **Goal:** Restructure without breaking consumers.
- **What to do:** Add or cite a project rule: e.g. “**Preserve public exports and HTTP routes**; internal rename only.” `@` the module and its tests.
- **Tip:** Ask for **mechanical commits** (rename vs behavior) if the diff grows large.

### Tests first, then implementation

- **Goal:** Lock behavior before refactoring or new features.
- **What to do:** `@` existing tests or the test folder. “Add failing tests for … then implement until green.”
- **Tip:** Specify **test framework** and **folder conventions** once in a rule so you do not repeat them. **Full playbook:** [write-test-case.md](write-test-case.md) (TDD, regression, contract, UI tests, prompts).

### Docs and decisions pass

- **Goal:** Keep README / ADRs aligned with code.
- **What to do:** `@` the changed code and `README` or `docs/`. “Update docs to match; no scope creep.”
- **Tip:** For ADRs, ask for **Decision / Context / Consequences** only—skip essay length.

### Ticket → implement → verify (with MCP)

- **Goal:** Close the loop from issue to proof.
- **What to do:** With a suitable MCP server (e.g. **issues** + **browser**), fetch the ticket acceptance criteria, implement, then **verify** in the app or staging URL.
- **Tip:** Paste the **exact URL** and test account constraints into the prompt; never put **passwords** in rules.

### Major dependency or framework upgrade

- **Goal:** Move a major version (runtime, framework, or key lib) without silently breaking the app.
- **What to do:** `@` `package.json` / lockfile / build config / entry docs. Ask for a **phased plan**: breaking changes from upstream, codemods, file-by-file checklist, test gaps, rollback.
- **Tip:** Ask for **mechanical steps first** (lint/tsconfig/build), then runtime fixes; commit after each phase so diffs stay reviewable.

### CI or build failure from logs

- **Goal:** Turn a red pipeline or local build into a minimal fix.
- **What to do:** Paste the **full failing log** (or link + excerpt). `@` the workflow file (e.g. `.github/workflows/…`), Dockerfile, or script named in the error.
- **Tip:** Say whether failure is **reproducible locally** with one command; ask the agent to **reproduce → fix → add guard** (e.g. stricter script or test).

### Pre-merge PR risk review

- **Goal:** Catch security, API, and data risks before merge when you already have a diff.
- **What to do:** `@` changed files or paste the PR description + key hunks. Ask for **risk list** (authz, injection, PII leaks, migrations, backwards compatibility), not generic praise.
- **Tip:** Add **“assume attacker on the same network / with a stolen cookie”** for API-facing changes to surface auth and validation gaps.

### Observability pass (logging / metrics / traces)

- **Goal:** Make production failures debuggable without inventing a new logging style per file.
- **What to do:** `@` existing observability helpers (logger module, OpenTelemetry setup, middleware). “Add structured context to this path: …; match existing patterns only.”
- **Tip:** Forbid **logging secrets** in rules or in the prompt; ask for **sample log lines** and **redaction** behavior.

### Security and hygiene pass (deps + secrets mindset)

- **Goal:** Reduce obvious foot-guns: leaked secrets patterns, risky deps, unsafe defaults—**without** claiming full pentest coverage.
- **What to do:** `@` dependency manifests and sensitive areas (auth, uploads, eval). Ask for **prioritized findings** and **small fixes**; run your real scanner/ Dependabot separately.
- **Tip:** Use **`.cursorignore`** so dumps and `.env` never enter context; describe env **variable names** only, not values.

### Localization (i18n) or accessibility (a11y) sweep

- **Goal:** Align strings and keyboard/ARIA behavior with product and compliance expectations.
- **What to do:** `@` UI package or routes. “List user-visible strings missing i18n keys” or “Check focus traps, labels, and heading order for these screens.”
- **Tip:** Pair with **one** reference component that “does it right” via `@` so the agent copies structure, not inventing a second pattern.

### Performance hot path (profile-guided)

- **Goal:** Improve latency or memory where metrics say it hurts—not random micro-optimizations.
- **What to do:** Paste **numbers** (trace id, slow query, flame graph summary) and `@` the suspected modules. Ask for **measured hypothesis** + change + how to verify.
- **Tip:** Require **before/after** verification steps (benchmark command, reproduction query)—skip changes that cannot be validated in-repo.

---

## Appendix: first-day checklist

- [ ] Workspace root is correct; **indexing** finished for the paths you edit.
- [ ] You tried **Chat** once and **Agent** once on a **small** task to learn the difference.
- [ ] You added at least one **project rule** stub under `.cursor/rules/` if the team shares conventions.
- [ ] You enabled **MCP** only for servers you need; **credentials** are scoped and not stored in rules.
- [ ] You **reviewed diffs** and ran **tests or the app** before your first merge.

---

_This page is a local index. Official reference: [cursor.com/docs](https://cursor.com/docs)._
