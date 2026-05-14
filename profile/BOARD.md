# BOARD.md — the HeddleCo kanban contract

Project board: [HeddleCo · Heddle](https://github.com/orgs/HeddleCo/projects/1)

This is the document a human or agent reads cold to learn how the board works: what every field means, how to claim a task, what "done" requires, and how cross-repo dependencies are tracked. It is the single source of truth for the board itself. The orchestrator-side script details live in `HeddleCo/CLAUDE.md` (private). The agent-side execution contract lives in `HeddleCo/AGENTS.md` (private). This file is the public contract that links the two.

If anything here disagrees with the issue you are working on, **the issue wins** — but please open a PR against this file in the same change so the next reader is not lied to.

---

## Repos under this board

The project tracks work across three repos plus this org-level one:

- `HeddleCo/heddle` (public) — the OSS Rust CLI and core crates.
- `HeddleCo/weft` (private) — the hosted server (control plane, gRPC, Postgres).
- `HeddleCo/tapestry` (private) — the SvelteKit web product on Cloudflare Workers.
- `HeddleCo/.github` (this repo) — org-level docs, profile README, and this board contract.

Issues live in whichever repo the work touches. Cross-repo work uses one issue per repo with `Blocked by` links — see [Cross-repo work](#cross-repo-work) below.

---

## Field meanings

The board has six configured fields beyond the GitHub-native title/assignee/labels.

### Status

The lifecycle column. Five values:

| Status | Meaning | Who moves it |
|---|---|---|
| **Backlog** | Filed but not yet ready to start. Blocked, under-specified, or low priority. | Default on issue creation. Anyone can move. |
| **Ready** | Acceptance criteria are clear, all `Blocked by` issues are closed, an agent could pick this up cold. | Maintainer triage. |
| **In progress** | Someone has claimed it and is actively working. Worktree exists. | The agent claims it (or the orchestrator). |
| **In review** | A PR is open referencing this issue. CI may be green or red. | Auto-set by the [project-card-sync workflow](#workflows-that-move-the-board) when a PR opens with `Closes` / `Part of` / `Refs`. |
| **Done** | The PR that closed this issue has merged into the default branch. | Auto-set by the project-card-sync workflow on merge. Manual edits will be overwritten. |

The board view groups by Status. The flow is **Backlog → Ready → In progress → In review → Done**, never backwards by hand. Reopening an issue resets it to Backlog (also automated).

### Epic

Which epic this sub-issue rolls up to. Ten epics exist:

| Epic | Theme |
|---|---|
| **E0 QA Blockers** | Three pre-existing import / replay / packfile bugs that block downstream epics. |
| **E1 Codebase Consolidation** | Collapse duplicated crates between heddle and weft; weft becomes server-only. |
| **E2 Self-Sovereign Auth** | Browser-held Ed25519 keys, client-side biscuit mint, no server-side root creds. |
| **E3 Heddle 0.3 Stability** | Transaction replay, partial clone, coverage gate, multi-OS FUSE spike. |
| **E4 Weft Hardening** | Observability, rate limits, multi-instance, deploy to `weft.heddle.sh`. |
| **E5 Tapestry Review MVP** | Threaded discussions, signature collection, file/line annotations on real APIs. |
| **E6 GitHub Mirror** | Post-receive push to GitHub; webhook ingest; UI mirror status. |
| **E7 Dogfood Cutover** | Sequenced M1–M4 migration of HeddleCo's own dev from GitHub to Weft. |
| **E8 Tapestry UX Depth** | Namespace tree, access debugger, real activity feed, command palette. |
| **E9 Docs & Launch** | Onboarding path, contribution guide, blog cadence, public launch. |

Use the Epic field to filter the board to one workstream. Closing the last sub-issue under an epic closes the epic itself (currently manual; automation TBD).

### Scope

Which repo(s) the change touches:

- **heddle** — only modifies files under `HeddleCo/heddle`.
- **weft** — only modifies files under `HeddleCo/weft`.
- **tapestry** — only modifies files under `HeddleCo/tapestry`.
- **multi** — touches two or more. Expect more than one PR. See [Cross-repo work](#cross-repo-work).

### DoD Type

What "done" looks like for this issue. Determines which sections the [DoD CI gate](#workflows-that-move-the-board) requires in the PR description:

| DoD Type | Required PR sections |
|---|---|
| **TDD-Rust** | `## Test evidence` (cargo test output) **and** `## Red commit` (SHA of the failing-tests-first commit). |
| **E2E-UI** | `## Test evidence` (Playwright output) **and** `## Manual verification` (viewports, before/after screenshots). |
| **Spike** | `## Test evidence` may be `N/A`. Deliverable is a decision doc — link it from the PR. |
| **Docs** | `## Test evidence` may be `N/A — pure docs, validated by visual inspection`. Manual verification optional. |
| **Ops** | Test evidence is the runbook output, terraform plan, deploy log, etc. Manual verification covers post-deploy checks. |

A linked-issue reference (`Closes`, `Fixes`, `Resolves`, `Part of`, `Tracked by`, `Refs`) is required on every PR regardless of DoD Type.

### Size

Rough estimate. Used for batching and stale-claim heuristics.

| Size | Rough scope | Stale-claim threshold |
|---|---|---|
| **XS** | Single file, < 30 min. | 2h without a commit. |
| **S** | A handful of files, < 2h. | 4h without a commit. |
| **M** | One subsystem, half a day. | 12h without a commit. |
| **L** | Cross-cutting, full day. | 24h without a commit. |
| **XL** | Multi-day or cross-repo epic-shaped work. | Re-scope; XL issues should split. |

### Priority

| Priority | Meaning |
|---|---|
| **P0** | Production / dogfood blocker. Drop everything. |
| **P1** | Current sprint. Default for new sub-issues. |
| **P2** | Important but not urgent. |
| **P3** | Nice to have. Eligible for "first contribution" labels. |

---

## How to claim a task

Two-step claim. The board does not assume the assignee field is the source of truth — Status is.

1. **Set the assignee to yourself** on the issue.
2. **Move Status to In progress** in the board view.

That's it. The orchestrator (`HeddleCo/scripts/dispatch.py`) will see the claim on its next sweep and skip the issue. If you are an agent, also post the claim comment per the [five-comment protocol](#the-five-comment-protocol) so any concurrent run knows your file scope.

Before you claim:

- Confirm `Status = Ready`. If it is `Backlog`, the issue is not yet accepted for work — comment asking for triage rather than claiming directly.
- Read every issue listed under the issue's `## Blocked by` section. Every one must be `Closed`. If any is open, do not claim — see [Cross-repo work](#cross-repo-work).
- Read the parent epic's `## Critical files` section. If your issue lacks a `## File scope`, infer one from the parent and post it as your first comment so the orchestrator can detect file-scope collisions with concurrent claims.

If you cannot proceed after claiming, post the BLOCKED comment and apply the `blocked` label. Do not silently un-claim.

---

## Definition of Done

Every sub-issue carries the same DoD shell at the bottom of its body:

```markdown
## Definition of Done
- [ ] **Red-commit first** (Rust): failing tests pushed before implementation; link the red SHA in the PR.
- [ ] **Playwright e2e + verification checklist** (UI): viewports, auth states, before/after screenshots in PR.
- [ ] PR description includes test command output + coverage delta + perf delta where relevant.
- [ ] Cross-repo PRs all linked back to this issue.
- [ ] Acceptance criteria above all checked.
```

Which lines apply depends on the issue's `DoD Type` field — see the table above. The DoD CI workflow (`.github/workflows/dod-check.yml` in each repo) is the enforcer; if it fails, the PR is not merge-ready.

The PR description must follow this shape:

```markdown
## Summary
<2–4 bullets describing the change>

## Closes
HeddleCo/<repo>#<n>          <!-- or `## Part of` for cross-repo PRs that don't directly close the issue -->

## Red commit
`<sha>`                       <!-- TDD-Rust only -->

## Test evidence
```
<cargo test / bun test / playwright report output>
```

## Manual verification
<UI: viewport list + before/after screenshot URLs. Other: 'N/A — covered by tests'>

## Blocked by → resolved
<If this PR resolves anyone else's `## Blocked by`, list those issues>
```

The DoD check runs on `pull_request_target` and posts a sticky comment with pass/fail. It will not auto-merge; a maintainer (or the `ship` flow) does that.

---

## Cross-repo work

Some work spans repos. The convention is **one issue per repo, linked by `Blocked by`**. Example: E2 (self-sovereign auth) has a `tapestry` issue, two `weft` issues, and a `heddle` doc issue, all linked.

### Filing cross-repo issues

- File the issue in the repo whose code you are touching. Do not file all sub-issues in `heddle` for tracking convenience — the project board aggregates them already.
- Set `Scope = multi` on the parent / coordinator issue and `Scope = <repo>` on each per-repo sub-issue.
- Use the `Blocked by` block to express ordering: a child blocked by a sibling will not move to `Ready` until the sibling closes.

```markdown
## Blocked by
- HeddleCo/weft#42      <!-- pubkey RPC must land first -->
- HeddleCo/heddle#19    <!-- doc example must be ready for tapestry to link to -->
```

`Blocked by` accepts both same-repo (`#42`) and cross-repo (`HeddleCo/weft#42`) refs. The board surfaces these in the standup roll-up.

### PRs for cross-repo work

When a single change requires more than one PR:

- **Use `Part of HeddleCo/<repo>#<n>`** in every PR description. This puts the PR in `In review` on the kanban without auto-closing the issue when one PR merges.
- **Use `Closes HeddleCo/<repo>#<n>`** only on the *last* PR in the chain — the one whose merge should close the issue. If unsure, use `Part of` everywhere and close the issue manually after the final merge.
- Mention sibling PRs in each PR's `## Summary` so reviewers can find the chain.

The project-card-sync workflow distinguishes closing refs (`Closes`, `Fixes`, `Resolves`) from tracking refs (`Part of`, `Tracked by`, `Refs`). Closing refs trigger Done on merge; tracking refs only move to In review.

---

## The five-comment protocol

Agents post **only** these five comment types on issues. No chatty progress, no "looking into X", no "trying Y next". Long-form context belongs in the PR.

| Event | Comment |
|---|---|
| **Claim** (you start) | `🤖 Starting. Worktree \`workspace/issue-<n>\`, branch \`task/<n>-<slug>\`. File scope: <list>.` |
| **Red commit** (Rust only) | `🔴 Red commit \`<sha>\` — failing tests: \`<test::names>\`.` |
| **Block** (cannot proceed) | `🚫 BLOCKED: <one-line reason>. Needs: <what would unblock>.` Apply `blocked` label. |
| **Budget** (token / time exceeded) | `💸 BUDGET EXCEEDED at <est tokens>. Worktree preserved at <path>. Awaiting human review.` |
| **Done** (PR merged) | (none — the project-card-sync workflow handles this on merge) |

Humans triaging on the board read these comments to see who has what. Anything else is noise.

---

## Workflows that move the board

Two workflows live in each of the three repos and are responsible for everything automated on the board.

### `.github/workflows/project-card-sync.yml`

Listens to issue and PR events; writes to project field values via `PROJECT_PAT`.

- New issue → added to project, `Status = Backlog`.
- Reopened issue → reset to `Status = Backlog`.
- PR opened / edited / synced / ready_for_review → all linked refs (closing + tracking) move to `Status = In review`.
- PR merged into the default branch → closing refs (`Closes` / `Fixes` / `Resolves`) move to `Status = Done`; tracking refs (`Part of` / `Tracked by` / `Refs`) stay at `In review`.
- PR closed without merge → closing refs move back to `Status = Backlog`.

Fork PRs are skipped (they cannot move the board) so external contributors cannot mutate kanban state by crafting refs.

### `.github/workflows/dod-check.yml`

Listens to PR events; lints the PR description against the DoD shape and posts a sticky pass/fail comment.

- Always required: a closing or tracking ref to the kanban issue.
- Required when the PR touches non-`.md` / non-`.github/` code: `## Test evidence` section with a fenced code block.
- Required when the PR touches `.rs` files: `## Red commit` section with a SHA.
- Required when the PR touches `.ts` / `.tsx` / `.svelte` / `.js` files (and no `.rs`): `## Manual verification` section.

Pure-docs and pure-CI PRs are exempt from the test-evidence requirement.

---

## Triage cadence

- **Daily standup** — orchestrator runs `scripts/standup.py` and surfaces stale claims, missing DoD evidence, and topology violations (sub-issues whose `Blocked by` siblings are still open but whose Status is Ready).
- **Weekly retro** — closed-this-week count by epic, PRs merged, agent vs. human commits.
- **Per-PR DoD check** — automated, on every PR event.

If you find a board entry in a state that violates this contract, comment on the issue rather than fixing it silently. The orchestrator's CLAUDE.md addresses how to release stale claims and reset bad state.

---

## Reading order for a new contributor

1. This file (BOARD.md) — board mechanics.
2. The repo you are working in: its `AGENTS.md` and `CLAUDE.md`.
3. The issue you are picking up — `## Acceptance criteria` plus parent epic's `## Critical files`.
4. The PR template (`.github/PULL_REQUEST_TEMPLATE.md`) before opening the PR.

If any step is unclear, comment on the issue. The orchestrator is reading.
