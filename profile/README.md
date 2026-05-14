<img src="https://heddle.sh/brand/github-banner.svg" alt="HEDDLE. — Version control for agent work." width="100%">

*Names, not noise. One task. One resolved state. Every change permanent, attributed, and inspectable.*

---

## What we make

Heddle is version control for agent work. Agents don't work in commits, so we stop versioning their work like humans do. Every agent task — captures, retries, aborts, conflicts, one merge — is bound into a single named, reviewable, recoverable thread. Drop it in beside Git. Keep Git. Add the missing control layer.

Built for teams with real agentic workflows: parallel agents, review bottlenecks, provenance requirements, or recovery pain.

## Projects

- **[`heddle`](https://github.com/HeddleCo/heddle)** — open source. The Rust CLI and core. Stable change IDs that survive history rewrites, explicit human and agent attribution on every state, and a Git-overlay mode so adoption costs you nothing on day one.
- **Hosted web app** — closed source. The product surface at heddle.sh. Repository inspection, review, and admin woven over the hosted control plane.
- **Hosted server** — closed source. The control plane behind the hosted app. Namespaces, repositories, grants, ingest, and review as first-class primitives.

## Try it

```bash
cargo install heddle-cli
heddle --help
```

## Status

Foundation in place. Local repository operations and Git interop are usable today; hosted surfaces are rolling out incrementally.

## Links

- Site — [heddle.sh](https://heddle.sh)
- Docs — [heddle.sh/docs](https://heddle.sh/docs)
- Brand — [heddle.sh/brand](https://heddle.sh/brand)
- X — [@HeddleThreads](https://x.com/HeddleThreads)
- Contact — [hello@heddle.sh](mailto:hello@heddle.sh)

---

*Made carefully.*
