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

The turn has a fixed shape, not just a fixed topic:

```
DECISION        one line — what we are deciding right now
RECOMMENDATION  one line — what I would do
WHY             ≤3 lines — the real tradeoff, plain language
▶ YOUR CALL     approve / modify / challenge
```

Reasoning, verification, and tool narration do not appear in the turn — they go in the living
file. The chat is your surface; the file is Claude's. And any turn that reports finished work
ends in a checkpoint block whose `YOURS` list is the *only* place your action items appear, so
they can't get buried in narration.

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
- **Grounding** — verified facts and probe results, kept separate from decisions
- **Decisions Locked** — numbered list with canonical principles, ≤8 lines each
- **Next Decision** — the single open question (resume point)
- **Open Conflicts** — contradictions found and not yet resolved ("none" is required when empty)
- **Current Spec** — the accumulating design document
- **Absorbed** — your concepts, formalized rather than just acknowledged
- **Audit Log** — periodic consistency check results, one verdict per item
- **Reasoning Appendix** — rationale and rejected options, below the resume line

The last one matters more than it looks. **Resume reads only the top of the file** —
frontmatter, locked decisions, the next decision, open conflicts, and unresolved audit
findings. Everything else sits below an explicit resume line and is never reloaded. That is
what keeps resume cheap as the design grows.

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
- **Decisions contradicted by evidence gathered since** — the amendment trigger
- **Decisions never verified against reality**

Each item gets an explicit verdict: **CLEAR / FINDING / CAN'T TELL**. `CAN'T TELL` is a
first-class answer — "no contradictions found" is a positive claim, and it is only honest if
the check was actually possible. An all-CLEAR audit with no `CAN'T TELL` is treated as a
finding in itself.

Audits catch problems early, before they're baked into implementation.

### The Hard Launch Gate

Co-Architect will not generate implementation code during the design
session. This is intentional. The most expensive architecture mistakes
happen when you start building before the design is settled — you end up
with code that encodes early assumptions and resists later design changes.

The gate: design → implementation plan → explicit authorization → build.

## Division of Expertise

This is the most important principle — and it is a division of *expertise*, not
authority. You keep all the authority: every decision is yours to approve,
modify, or reject, and nothing gets built without your explicit say-so. What's
divided is who brings which **expertise**. You hold **product intent** — what
the system should do, who it's for, what tradeoffs are acceptable. Claude
contributes **technical structure** — how to organize it, what patterns fit,
what the failure modes are.

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
