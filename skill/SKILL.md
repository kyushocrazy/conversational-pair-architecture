---
name: co-architect
description: >-
  Conversational Pair Architecture — a collaborative, one-decision-at-a-time
  design dialogue that produces an evidence-gated spec and resumes across
  sessions and devices. Use when the user wants to design or architect something
  through back-and-forth rather than a one-shot plan; when they say "let's
  co-architect", "pair-architect this", "design this with me"; or when a design
  session needs to continue later. Maintains a living session file for
  cross-session resume.
---

# Conversational Pair Architecture (CPA)

Run a collaborative design dialogue with the user: one meaningful decision at a time,
recommendation-first, each locked decision captured as a canonical principle, all recorded
in a living session file that can resume across sessions and devices.

## When to use
- The user opens with "let's co-architect …", "pair-architect this", "design this with me",
  or invokes `/co-architect`.
- A design task that benefits from progressive, decision-by-decision elaboration.
- Any design the user may continue later or from a different device.

## The core principle

> **The chat turn is the user's surface. The living file is Claude's.**

If the user needs it to decide, it goes in the turn. Reasoning, verification, tool narration,
and recaps go in the living file. Nothing else appears in the turn. This is a routing rule,
not "be brief" — reason and push back as hard as ever, in the file.

## The turn contract

Every turn that opens a decision emits exactly this — **≤12 lines total**, blank lines and
`▶ YOUR CALL` included, **no line past ~2 sentences**. `▶ YOUR CALL` carries ≤3 items:

```
DECISION        one line — what we are deciding right now
RECOMMENDATION  one line — what I would do
WHY             ≤3 lines — the real tradeoff, plain language
▶ YOUR CALL     approve / modify / challenge (+ anything genuinely the user's)
```

**A modification is not an approval.** Restate the decision with the modification folded in,
one line, and lock only after the user confirms that restatement. Never lock your own reading
of a modification.

Any turn that **reports completed work** — and session end, handoff, or "where are we" — ends
in the checkpoint block and nothing else:

```
DONE      what is locked and verified
NOT DONE  what is open — anything skipped or blocked goes FIRST
YOURS     numbered list — ONLY things the user must personally do or decide
NEXT      the single next decision
```

**Budget: one line per item, ≤4 items per section.** Overflow goes to the living file. No
paragraphs anywhere in this block — if `DONE` needs a paragraph, the paragraph belongs in
the file.

**Every ask is one numbered item in `YOURS`. There is one list.** `DONE`, `NOT DONE`, and
`NEXT` are context and never carry the ask.

`references/interaction-contract.md` is authoritative and carries the full exclusion list, the
lock / challenge / modify / question-mid-decision turn shapes, the living-file budget, audit
verdicts, amendments, and the probe exception. Read it at session start.

## How to run
Narrow to one decision → recommend with the tradeoff → approve / modify / challenge → **on
approval**, lock with a canonical principle → record in the living file → advance. Audit every
~5 decisions, one verdict per checklist item in `## Audit Log`; the turn shows only `FINDING` /
`CAN'T TELL` plus a count of the `CLEAR`s. Implement nothing until the launch gate (design →
plan → authorization), except a probe named in the turn before it is written.

## Starting a session
1. Ask ONE thing: "What are we designing, and where does it eventually live?"
2. Read the session index. If any `active` session has not been updated in 30+ days, say so
   in one line: *"N sessions have been idle 30+ days — close or abandon?"* It is a flag, not
   an agenda — do not expand on it.
3. Resolve the living-file home (see "Living-file home resolution").
4. Create the living file from `references/session-template.md`, slug = kebab of the title.
5. Add a row to the session index (see `references/index-template.md` for the format).

## Resuming a session
- On a bare opener with no target, read the session index, list open sessions, and ask
  resume-or-new.
- On resume: pull the home repo first (if it is a git repo), then read **only the resume
  line**: frontmatter, `## Decisions Locked`, `## ▶ Next Decision`, `## Open Conflicts`, and
  unresolved `## Audit Log` findings. Never `## Reasoning Appendix`. Never
  `## Current Spec (living)` unless the open decision touches it.
- Announce "Resuming **<title>**. Last locked: Decision N. Next up: <Next Decision>." and
  continue the loop.

## Session status vocabulary
The index `Status` cell is **exactly one word**:

`active | paused | design-complete | handed-off | abandoned`

Detail — which decisions locked, where the spec went, what is next — belongs in the living
file, never in the index cell. Keep the table unbroken: a blank line between rows splits it
into two tables.

## Living-file home resolution
Read `references/homes.json` (if configured). Determine the destination project.
- If it has an entry with `cross_device: true` → home the living file at that entry's `path`.
- Otherwise (bare repo, no multi-device support, or unregistered) → fall back to a `sessions/`
  directory in the current project root, and record the intended destination in the living
  file's `destination:` frontmatter.
- `spec_destination`, when present, is where the **finished spec** is routed at handoff. It is
  not a living-file home and does not need `cross_device`.

Always update the index with the resolved home path, in repo-relative form (`sessions/`) —
absolute machine-specific paths do not resolve on another device.

## Continuity rules (correctness-critical)
- Pull the home repo on resume (if it is a git repo).
- After EVERY locked decision and EVERY audit: write the living file, refresh `updated`, and
  commit (and push) the home repo. The other device only sees what is committed.

### ⛔ Branch guard — commit the living file ONLY to the home branch (`main` or `master`)
**This is the #1 cross-device continuity failure.** Mobile/cloud Claude Code sessions can
silently auto-create an ephemeral branch (`claude/co-architect-xxxx`) and commit there. The
resume flow only reads the home branch, so the living file vanishes from resume and needs a
manual fast-forward to recover.

**Before EVERY commit/push of the living file, you MUST be on the home branch.** Never commit
the living file to a `claude/*` (or any other ephemeral session) branch.

```bash
# Run this in the home repo before each living-file commit:
branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$branch" != "main" ] && [ "$branch" != "master" ]; then
  git stash --include-untracked        # park the living-file edit
  git checkout main                    # or master — your home branch
  git pull --ff-only                   # reconcile with the other device
  git stash pop                        # reapply the edit on the home branch
fi
git add <living-file>
git commit -m "CPA: <what changed>"
git push                               # ALWAYS the home branch — never the session branch
```

If a previous turn already committed the living file to a `claude/*` branch, recover it:
`git checkout main && git pull --ff-only && git cherry-pick <that-commit>` (or fast-forward
the home branch to it), then push. Then delete the stray branch so it can't be resumed from.

## Completion handoff
When the design is approved and locked:
1. Set the living file `status: handed-off` and fill its Handoff section.
2. Route the final spec to the registry entry's `spec_destination` (or the destination
   project's docs).
3. Move or delete any probe code — it never stays in `sessions/`.
4. Offer to generate an implementation plan.

## Files
- `references/interaction-contract.md` — the dialogue discipline (authoritative)
- `references/session-template.md` — living session file template
- `references/index-template.md` — index row format + seed
- `references/homes.json` — project → living-file-home registry (user-configured; copy
  `references/homes.json.example` to start)
