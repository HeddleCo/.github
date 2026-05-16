<img src="https://heddle.sh/brand/banner-github.svg" alt="HEDDLE. — Version control for agent work." width="100%">

*Names, not noise. One task. One resolved state. Every change permanent, attributed, and inspectable.*

---

## What we make

**Heddle** is version control for agent work. Agents don't work in commits, so we stop versioning their work like humans do. Every agent task — captures, retries, aborts, conflicts, merges — is bound into a single named, reviewable, recoverable thread. Drop it in beside Git. Keep Git. Add the missing control layer.

Built for teams with real agentic workflows: parallel agents, review bottlenecks, provenance requirements, or recovery pain.

## Why a new layer

Git was designed for humans committing in serial. Agents work differently: they branch eagerly, retry on failure, abandon work mid-flight, and produce volumes of small changes that drown the review queue. The usual response — squashing, rebasing, force-pushing — destroys the provenance you need to debug a misbehaving agent or audit a decision after the fact.

Heddle keeps Git as the substrate and adds the missing layer on top: a *task* is the unit of work, every state on it is named and durable, and rewinding or replaying an attempt doesn't lose the trail. You can adopt it without migrating off your existing tooling — it overlays your repo today and you keep your `main` branch exactly as it is.

## Repos

The product is split across three repositories, plus this org-level one.

### [`heddle`](https://github.com/HeddleCo/heddle) — public, Apache-2.0

The Rust CLI and core crates. This is the OSS engine of the product: stable change IDs that survive history rewrites, explicit human and agent attribution on every state, and a Git-overlay mode so adoption costs you nothing on day one. Runs locally; talks to a hosted control plane when you want one.

Install with `cargo install heddle-cli`. See [the repo README](https://github.com/HeddleCo/heddle#readme) for the user-facing walkthrough, and [CONTRIBUTING.md](https://github.com/HeddleCo/heddle/blob/main/CONTRIBUTING.md) for the reading order before your first PR.

### `weft` — private

The hosted control plane behind heddle.sh: namespaces, repositories, grants, ingest, and review as first-class primitives. Where shared state lives when a team adopts the hosted product, and where multi-tenant policy and quota are enforced. Closed source today; some of its primitives may be split out as the boundary with `heddle` stabilizes.

### `tapestry` — private

The web product surface at [heddle.sh](https://heddle.sh). Repository inspection, review queues, and admin woven over the hosted control plane. SvelteKit on Cloudflare Workers; talks to `weft` over the same RPC surface that `heddle` does.

### [`HeddleCo/.github`](https://github.com/HeddleCo/.github) — public

This repo. Org profile (the page you're reading), and over time the default community health files and shared reusable workflows that the three product repos call into.

## Try it

```bash
cargo install heddle-cli
heddle --help
```

For walkthroughs and integration notes, head to [heddle.sh/docs](https://heddle.sh/docs).

If you want to read code first, start in [`heddle/crates`](https://github.com/HeddleCo/heddle/tree/main/crates) — the crate names map directly to the concepts in the docs (changes, tasks, attribution, overlay) and most have their own README with a short tour.

## How we work

Development across all three repos is tracked on a single org-wide kanban: [HeddleCo · Heddle (Project #1)](https://github.com/orgs/HeddleCo/projects/1). Issues are the unit of work; every PR closes one with `Closes HeddleCo/<repo>#<n>` (or `Part of` for cross-repo work).

A few things worth knowing if you want to contribute:

- **Issues drive PRs, not the other way around.** A claimable issue has clear acceptance criteria and no open blockers. If either is missing, file a comment on the issue rather than guessing intent and opening a speculative PR.
- **Each repo has its own `CONTRIBUTING.md`** with build/test commands and reading order. Those take precedence over anything at this org level.
- **External contributions are welcome on the public repos.** `heddle` and this `.github` repo are open to outside PRs; `weft` and `tapestry` are private and not currently accepting them.
- **Security issues should not be filed publicly.** A coordinated disclosure channel is in flight — until it lands, please reach out via the contact link below and we'll route it.

## Status

Foundation in place. Local repository operations and Git interop in `heddle` are usable today; the hosted control plane and web product surfaces are rolling out incrementally. See per-repo READMEs and CHANGELOGs for the authoritative current state.

What's stable:

- `heddle` CLI for local task tracking on top of an existing Git repo
- Change-ID stability across history rewrites (rebase, amend, squash)
- Human/agent attribution captured on every state transition
- Git-overlay mode — no separate storage, no rewrites of `main`

What's in flight:

- Hosted review surfaces in the web app
- Multi-tenant control plane in `weft`
- Reusable workflow library for downstream Rust projects

The kanban linked above is the source of truth for what's next. Things move quickly, so the per-repo CHANGELOG is the surest way to see what's landed in the last week.

## Links

- Site — [heddle.sh](https://heddle.sh)
- Docs — [heddle.sh/docs](https://heddle.sh/docs)
- Brand — [heddle.sh/brand](https://heddle.sh/brand)
- Kanban — [HeddleCo · Heddle](https://github.com/orgs/HeddleCo/projects/1)
- X — [@HeddleThreads](https://x.com/HeddleThreads)
- Contact — [hello@heddle.sh](mailto:hello@heddle.sh)

---

*Made carefully.*
