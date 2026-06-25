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

Run a collaborative design dialogue with the user: one meaningful decision at a
time, recommendation-first, each locked decision captured as a canonical
principle, all recorded in a living session file that can resume across sessions
and devices.

## When to use
- The user opens with "let's co-architect …", "pair-architect this", "design
  this with me", or invokes `/co-architect`.
- A design task that benefits from progressive, decision-by-decision elaboration.
- Any design the user may continue later or from a different device.

## How to run
Follow `references/interaction-contract.md` exactly — it is the discipline.
Summary: narrow to one decision → recommend with tradeoff → approve / modify /
challenge → lock with a canonical principle → record in the living file →
advance. Audit every ~5 decisions. Implement nothing until the launch gate
(design → plan → authorization).

## Starting a session
1. Ask ONE thing: "What are we designing, and where does it eventually live?"
2. Resolve the living-file home (see "Living-file home resolution").
3. Create the living file from `references/session-template.md`, slug = kebab
   of the title.
4. Add a row to the session index (see index-template.md for format).

## Resuming a session
- On a bare opener with no target, read the session index, list open sessions,
  and ask resume-or-new.
- On resume: read the living file, then announce "Resuming **<title>**. Last
  locked: Decision N. Next up: <Next Decision>." and continue the loop.
- If the session file lives in a git repo, pull before reading to get the
  latest state from other devices.

## Living-file home resolution
Read `references/homes.json` (if configured). Determine the destination project.
- If it has an entry with `cross_device: true` → home the living file at that
  entry's `path`.
- Otherwise → fall back to a `sessions/` directory in the current project root.
- Record the intended destination in the living file's `destination:` frontmatter.
Always update the index with the resolved home path.

## Continuity rules (correctness-critical)
- Pull before resuming if the session file is in a git repo.
- After EVERY locked decision and EVERY audit: write the living file, refresh
  `updated`, and commit (and push if configured). The other device only sees
  what is committed.

### ⛔ Branch guard — commit the living file ONLY to the home branch
This is the #1 cross-device continuity failure. Mobile/cloud Claude Code
sessions can silently auto-create an ephemeral working branch
(e.g. `claude/co-architect-xxxx`) and commit there. The resume flow reads the
home branch (`main`/`master`), so the living file vanishes from resume and needs
a manual fast-forward to recover.

**Before every living-file commit, confirm you are on the home branch.** If not:
stash the edit, check out the home branch, pull, reapply, then commit and push.

```bash
branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$branch" != "main" ] && [ "$branch" != "master" ]; then
  git stash --include-untracked
  git checkout main            # or master — your home branch
  git pull --ff-only
  git stash pop
fi
git add <living-file>
git commit -m "CPA: <what changed>"
git push                       # always to the home branch, never a session branch
```

If a previous turn already committed to a stray branch, recover it:
`git checkout main && git pull --ff-only && git cherry-pick <that-commit>`, push,
then delete the stray branch so it can't be resumed from.

## Completion handoff
When the design is approved and locked:
1. Set the living file `status: complete` and fill its Handoff section.
2. Route the final spec to the destination project.
3. Offer to generate an implementation plan.

## Files
- `references/interaction-contract.md` — the dialogue discipline (authoritative)
- `references/session-template.md` — living session file template
- `references/index-template.md` — index row format + seed
- `references/homes.json` — project → living-file-home registry (user-configured)
