# How Co-Architect Works

## The Problem

AI-assisted architecture sessions have two common failure modes:

1. **The monologue.** You describe what you want, the AI dumps a complete
   architecture doc on you. It's plausible-looking but reflects the AI's
   assumptions, not your judgment. You read it, feel vaguely unsatisfied,
   and either accept it as-is or start over.

2. **The wandering chat.** You go back and forth with the AI in freeform.
   Good ideas emerge but nothing is captured. By message 30, neither of you
   remembers what was decided in message 8. The chat becomes the spec, and
   chat is a terrible spec format.

## The Solution: Per-Decision Architecture

Co-Architect enforces a structured loop that prevents both failure modes:

### One Decision at a Time

Instead of producing a complete architecture, Claude narrows to the single
most important open question and focuses the entire turn on it. This keeps
each interaction small enough to reason about carefully — especially
important on mobile, where long messages are hard to read and respond to.

### Recommendation-First

Claude doesn't ask open-ended questions like "how should we handle auth?"
That puts the burden on you to generate options from scratch. Instead,
Claude leads with a specific recommendation and the key tradeoff: "I'd
recommend JWT with short-lived tokens — simpler infra than sessions, but
you'll need a refresh token flow." You react to a concrete proposal, which
is faster and produces better decisions.

### Canonical Principles

When you approve a decision, Claude doesn't just move on. It translates
the decision into an explicit, quotable **canonical principle** — a named
rule that governs the rest of the design. "Decision 3 locked. **Principle:
All auth state lives in the token; the server is stateless for auth
purposes.**"

This matters because:
- Later decisions can reference earlier principles by number
- Audits check for principle contradictions
- The principles become the backbone of the final spec
- They're grep-able in the session file

### Living Session Files

All state lives in a Markdown file, not in the chat. The file has a
structured format:
- **Decisions Locked** — numbered list with canonical principles
- **Next Decision** — the single open question (resume point)
- **Current Spec** — the accumulating design document
- **Audit Log** — periodic consistency check results

This means:
- Sessions survive context window limits
- You can resume from a different device
- The file IS the deliverable — no separate "write up the spec" step
- Git tracks the evolution of the design

### Design Audits

Every ~5 decisions, Claude runs a systematic audit:
- **Contradictions** between locked principles
- **Missing owners** — who is responsible for each component?
- **Authority leaks** — does any component reach outside its boundary?
- **Hidden assumptions** — what are we taking for granted?
- **Unsafe defaults** — what fails dangerously if unconfigured?
- **Unhandled failure states** — what breaks and what happens when it does?

Audits catch problems early, before they're baked into implementation.

### The Hard Launch Gate

Co-Architect will not generate implementation code during the design
session. This is intentional. The most expensive architecture mistakes
happen when you start building before the design is settled — you end up
with code that encodes early assumptions and resists later design changes.

The gate: design → implementation plan → explicit authorization → build.

## Division of Authority

This is the most important principle. You hold **product intent** — what
the system should do, who it's for, what tradeoffs are acceptable. Claude
holds **technical structure** — how to organize it, what patterns fit, what
the failure modes are.

Neither substitutes for the other. Claude won't override your product
judgment with a "better" technical choice. You won't need to specify
implementation details you don't care about. The skill explicitly treats
your lived experience, taste, and workflow preferences as real design
evidence, not soft requirements to be rationalized away.

## Cross-Device Resume

The living session file is plain Markdown in git. The resume flow:

1. Pull the latest from the repo
2. Read the session index to find open sessions
3. Open the living file
4. Read the `▶ Next Decision` section
5. Announce: "Resuming **<title>**. Last locked: Decision N. Next up: ..."
6. Continue the per-decision loop

This makes mobile-to-desktop workflows seamless. Start a design session on
your phone during a commute, push the session file, then continue from your
desktop with full context.
