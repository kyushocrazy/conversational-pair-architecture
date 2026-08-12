# Open Items

Things logged for future follow-up. Not urgent, no assigned date.

---

## Codex evidence artifacts

The public `evidence.md` is Claude-centric. We owe a Codex complement:

- Transcripts from Codex CPA sessions
- Usage efficiency data (within-session tokens, resume cost)
- A/B comparisons similar to what we ran on Claude

Format doesn't need to be a dissertation — the framing we have in mind: a loose A/B across one of our own active projects, maybe a screen-record. README already names this as on the roadmap. When ready, fold into `docs/evidence.md` or add a `docs/evidence-codex.md`.

---

## Skills dependency audit

CPA was built and tested on one developer's setup, which has a large installed set of Claude Code skills and MCP tools. The co-architect skill may call — or implicitly assume — skills/tools that a fresh install won't have.

Questions to answer before calling this production-ready for outside users:

- Which skills does `co-architect` invoke (directly or by recommendation) during a session?
- Which MCP tools does it lean on?
- What happens if those aren't installed — graceful degradation or silent failure?
- Which are hard dependencies vs. soft "works better with"?

Document this in `docs/configuration.md` (or a new `docs/dependencies.md`) so users know what to install. Also flag anything we just take for granted that isn't in the public repo.

*Logged 2026-06-26 — came up because the workflow has only been tested on one setup.*
