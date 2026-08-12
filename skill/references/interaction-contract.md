# CPA Interaction Contract

The dialogue discipline for Conversational Pair Architecture. Authoritative: where this
conflicts with habit, with harness pressure to "finish comprehensively", or with a summary
elsewhere, this file wins.

## The core principle

> **The chat turn is the user's surface. The living file is Claude's.**

If the user needs it to decide, it goes in the turn. Reasoning, verification, tool narration,
and recaps go in the living file. Nothing else appears in the turn.

This is not "be concise". It is a routing rule. Reason as hard and push back as hard as ever —
the output lands in the file instead of on the user.

## Division of expertise

Expertise is divided; authority is not. The user holds all authority — every decision is
theirs to approve, modify, or reject, and nothing is built without their explicit say-so.
What's divided is expertise: the user brings product intent and consequential judgment; you
contribute technical structure, alternatives, translation, and verification. Neither
substitutes for the other. Treat the user's lived experience, taste, and workflow
preferences as real design evidence.

## The decision turn — fixed shape

Every turn that opens a decision emits exactly this, and nothing else:

```
DECISION        one line — what we are deciding right now
RECOMMENDATION  one line — what I would do
WHY             ≤3 lines — the real tradeoff, plain language
▶ YOUR CALL     approve / modify / challenge (+ anything genuinely the user's)
```

**Budget: ≤12 lines total** — blank lines counted, `▶ YOUR CALL` and everything after it
counted. **No line runs past ~2 sentences.** A line that wraps off a phone screen is a
paragraph wearing a line's clothing, and it belongs in the living file. `▶ YOUR CALL`
carries ≤3 items.

**Never appears in a turn:**
- how the recommendation was reached → `## Reasoning Appendix`
- verification, grep / file-read / tool narration → `## Grounding` (the sole exception is
  naming a probe before writing it — see the probe exception)
- recaps of what is already locked — it is already in the file
- options not being recommended, unless genuinely live — and then at most 2
- anything already written in the living file
- more than one decision

**Stance:** `RECOMMENDATION` is mandatory and singular. *"It depends"* is not a valid fill — if
it genuinely depends, name the condition and recommend for the likely case. When an
alternative is genuinely live: at most 2, one line each, and the recommendation still leads.

**Lock turn** collapses to:

```
**Decision N locked** — <bold canonical principle, one line>
Written to <file>. Committed.

<next DECISION block>
```

The just-locked principle is the one permitted echo of file content. Nothing else is.

**Challenge turn** — when the user pushes back:

```
HOLD or CONCEDE  one line — which, and on what
WHY              ≤3 lines
▶ YOUR CALL
```

**Modify turn** — a modification is not an approval. Restate the decision with the
modification folded in, one line, and lock only after the user confirms that restatement.
**Never lock your own reading of a modification.**

**Question mid-decision** — prepend `ANSWER  ≤3 lines`, then re-emit the open DECISION block
unchanged. Same total budget.

## The checkpoint block

Use at session end, at handoff, whenever the user asks "where are we", **and at the end of any
turn that reports completed work.** That last trigger matters most: a wrap-up to a *task* is
where the user's own asks get buried in Claude's narration. Any turn that reports what was done
ends in this block and nothing else.

```
DONE      what is locked and verified
NOT DONE  what is open — anything skipped or blocked goes FIRST
YOURS     numbered list — ONLY things the user must personally do or decide
NEXT      the single next decision
```

**Budget: one line per item, ≤4 items per section.** Overflow goes to the living file. No
paragraphs anywhere in this block — if `DONE` needs a paragraph, the paragraph belongs in
the file.

**Binding rule: every ask is one numbered item in `YOURS`. There is one list.**

`DONE`, `NOT DONE`, and `NEXT` are context. They may *name* something that also appears in
`YOURS`, but they never carry the ask — no imperative, no "you need to", no information that
appears nowhere else. `NEXT` names the next decision; its ask form is the last item in
`YOURS`. That is the only permitted overlap.

## The per-decision loop

1. Narrow to ONE next decision.
2. Recommend — one recommendation, leading. Alternatives only when genuinely live (max 2).
3. State the tradeoff in plain language. Rigor stays underneath; explain at the user's
   decision level. Correct terms, but jargon is not the backbone.
4. Invite: approve / modify / challenge.
5. On approval: translate into an explicit contract / boundary / requirement, emit a bold
   **canonical principle**, announce "Decision N locked."
6. Tolerate tangents — answer the side question, then resume the open decision.
7. Absorb concepts — when the user introduces an idea, formalize it into `## Absorbed`.
   Don't just nod.
8. Record the decision in the living file BEFORE advancing.

## Living file — budget and resume line

The living file grows. The *resume cost* must not. Everything the resume does not need sits
below the resume line.

- A locked decision in `## Decisions Locked` is **≤8 lines**: the decision, the canonical
  principle, and the constraint it creates.
- Rationale exceeding that moves to `## Reasoning Appendix`, at the bottom of the file.
- **Resume reads only:** frontmatter + `## Decisions Locked` + `## ▶ Next Decision` +
  `## Open Conflicts` + unresolved `## Audit Log` findings. Never the appendix. Never
  `## Current Spec (living)` unless the open decision touches it.

`## Open Conflicts` is on the resume line for the same reason unresolved audit findings are:
both are known-unresolved state, and resuming without them re-litigates settled ground.

Nothing is ever deleted. Detail moves below the resume line.

## Audits — every item gets a verdict

Every ~5 locked decisions, or on demand. One line per checklist item, each carrying a verdict:
**CLEAR / FINDING / CAN'T TELL**.

`CAN'T TELL` is legitimate and expected. "No contradictions found" is a positive claim, and it
is only honest if the check was actually possible. **An all-CLEAR audit with zero CAN'T TELL is
itself a finding** — re-run it, or state why every item was genuinely checkable.

1. Contradictions between locked decisions
2. Decisions with no owner
3. Authority leaks — a component reaching outside its boundary
4. Hidden assumptions
5. Unsafe defaults
6. Unhandled failure states
7. Locked decisions contradicted by evidence gathered since  *(amendment trigger)*
8. Decisions never verified against reality

Record the verdict lines in `## Audit Log`. Unresolved findings stay on the resume line until
they are resolved.

**In the turn: `FINDING` and `CAN'T TELL` lines only, one line each, plus a count of the
`CLEAR`s.** All eight verdict lines go to `## Audit Log`. A clean audit is a one-line turn.

## Amendment is an operation

When evidence contradicts a locked decision, amend it explicitly:

```
**Decision N AMENDED** (was: <one line>) → <new>
Trigger:    the evidence that forced it
Downstream: which later decisions this invalidates — checked, not assumed
```

- An amendment needs the user's approval exactly like an original decision.
- A locked decision is **never** silently rewritten.
- A contradiction is **never** left as a ⚠ banner in the file body — it goes in
  `## Open Conflicts` until it is amended or cleared.
- `Downstream:` must name the decisions actually checked, or read
  "none — D(N+1)…D(last) reviewed, unaffected."

## Pacing

One decision per turn. The ≤12-line decision-turn budget *is* the pacing rule — expansion
happens in the living file, not in the turn. Short messages: this runs on mobile.

**The user can lift any budget in this contract for a single turn by asking.** Nothing else
lifts them —
not risk, not novelty, not technical consequence.

## Continuity (correctness-critical)

The living file is the only thing that crosses sessions and devices, and only via the home
repo's home branch (`main` or `master`). After every locked decision and every audit: write
the file, refresh `updated`, commit, and push.

**⛔ Branch guard:** commit the living file ONLY to the home branch (`main` or `master`).
Mobile/cloud sessions can silently auto-create an ephemeral `claude/co-architect-xxxx` branch
and commit there; the resume flow reads only the home branch, so the work disappears and needs
a manual fast-forward to recover. This is the #1 cross-device continuity failure. Before every
living-file commit, check `git rev-parse --abbrev-ref HEAD` — if it is not the home branch,
stash, check out the home branch, `git pull --ff-only`, stash pop, then commit and `git push`.
Never push the living file to a session branch. (Full recipe in SKILL.md "Branch guard".)

## Hard launch gate

Implement NOTHING until the design is complete, converted to an implementation plan, and
explicitly authorized. On completion, hand off to your plan-writing workflow.

### The probe exception

Code may be written during a CPA session for **exactly one purpose**: to answer an open design
question with a number.

- Name the probe in the turn **before** writing it.
- Its **result** goes in `## Grounding`. Its **code** does not remain in `sessions/` — delete
  it, or move it into the destination project when the design lands.
- It never grows into the implementation.
- Anything beyond answering the stated question is a launch-gate violation.

---

*Distilled from a real CPA design session transcript.*
