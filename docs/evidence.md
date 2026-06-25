# Does It Actually Work? — A/B Evidence

Co-Architect makes a specific claim: that turning design into a structured,
one-decision-at-a-time dialogue produces **better-owned decisions, catches more
errors, and costs far less to resume** than a one-shot "give me the architecture"
prompt. This page is the honest evidence for that claim.

It is **not a dissertation.** Two small A/B runs, one subject each, measured
carefully and reported with their caveats. Directional, not conclusive — but the
core findings replicated across both rounds, which is why they're here.

## The setup

Each round runs the **identical brief** through two arms:

- **Arm A — one-shot (control):** a fresh, isolated session is handed the brief
  and asked to produce a complete design document in a single pass. No dialogue.
- **Arm B — Co-Architect (treatment):** the same brief, run through the
  per-decision CPA loop, with the human driving product/taste calls and Claude
  driving the technical lane.

Then we compare the durable artifacts, the token cost, and — the part that
matters most — what it costs to *resume the work later*.

| Round | Subject | What it stressed |
|---|---|---|
| **1** | Productizing **Open Notebook Desktop** (a desktop app: packaging, auto-update, code signing, telemetry opt-in, plus product identity) | Qualitative findings + the resume-cost argument. Numbers came out muddy (see caveats). |
| **2** | A **Skills Portal** — a tool that converts an agent/skill definition from one AI coding assistant's format to another's (later named *Agent Skills Translator*) | A clean control number, and an *actual* cross-device resume rather than a simulated one. |

Round 2 exists because round 1's within-session token numbers were contaminated
(Arm B was deliberately incomplete, and its transcript was inflated by an
unrelated troubleshooting detour). Round 2 tightened the protocol: a narrow
brief, both arms run to completion, no detours in the control.

---

## Quantitative

### Durable-artifact size (measured both rounds)

The thing you keep and act on afterward — the one-shot's document vs. CPA's
living session file.

| | Arm A one-shot doc | Arm B living file |
|---|---|---|
| **Round 1** | 2,683 words · ~4,450 tokens | 1,670 words · ~3,100 tokens |
| **Round 2** | 2,670 words · ~4,765 tokens | 1,942 words · ~3,575 tokens |

*(Token figures are char/4 approximations.)*

**Finding (replicated):** the CPA living file is **~25–30% smaller** than the
one-shot dump, while encoding *more usable structure* — every decision is
explicitly owned, carries a named canonical principle, and the file includes an
audit log. The one-shot is undifferentiated prose that includes choices the user
never made (and, in both rounds, at least one the user actively rejected — see
Qualitative).

### Within-session generation cost (clean control, round 2)

| Arm | Within-session tokens | How measured |
|---|---|---|
| **A (one-shot)** | **56,622** | harness usage meter on the isolated session (authoritative) |
| **B (CPA)** | split across two device-sessions — not a single number | needs a token-count over both transcripts to total |

Two honest notes here:

1. **CPA does not win on raw within-session tokens, and we don't pretend it
   does.** A multi-turn dialogue generates more conversational tokens than a
   single dump. For a quick, throwaway task, the one-shot is cheaper. We say so
   plainly.
2. That 56,622 is mostly fixed overhead — roughly 88% of it is the injected
   system prompt on a single turn — which is a reminder that raw token totals are
   a poor proxy for "cost of the design." The number that actually compounds in
   real work is the one below.

> Method note: the isolated one-shot session *self-estimated* ~41–45k tokens; the
> harness meter said 56,622. Trust the meter, not the model's self-report.

### Resume cost — the headline metric

This is what CPA is actually for. To pick the work back up later — tomorrow,
next week, or on a different device — how much context do you have to reload?

- **Co-Architect:** you reload **only the living file** — ~3,100 tokens (round 1),
  ~3,575 tokens (round 2). And that size is **roughly flat** no matter how long
  the dialogue ran.
- **One-shot / freeform chat:** to resume *with context*, you carry the **entire
  transcript** — easily **20,000–50,000+ tokens, and growing every single turn.**
- **Ratio:** order **~7–10× cheaper to resume**, and the gap **widens over time**
  (the living file stays roughly constant; a transcript grows linearly).

**In round 2 this was real, not simulated.** The session ran its first three
decisions on a desktop, then was **resumed on a phone**, which pulled the flat
~3,575-token living file and drove the remaining six decisions to completion.
Round 1 could only argue the mechanism; round 2 executed it.

---

## Qualitative — the real differentiator

Numbers aside, the clearest difference between the two arms shows up in *who
decides what.*

### 1. Division of expertise (you keep the authority)

A note on the name: the human keeps *all* the authority here — every decision is
theirs to approve, modify, or reject. What differs between the two arms is how
each respects the **expertise boundary**: whose call a given decision actually is.

In both rounds the one-shot quietly seized decisions that belong to the human:

- **Round 1:** it unilaterally invented a product name ("Lumen Notebook") and
  volunteered a full brand identity, palette, and voice — none of it requested,
  all of it in the owner's taste lane.
- **Round 2:** it named the product ("Rosetta") *and* invented a three-tier
  Free/Pro/Team pricing model — again, neither requested.

Co-Architect did the opposite. It **handed the naming decision back to the human**
both times, and in round 2 the human rejected several names for real collisions
(two candidates conflicted with an existing tool and a registered company) before
landing on a deliberate descriptive choice — collision diligence the one-shot's
"Rosetta" never did. On pricing, CPA *recommended* a paid seam and the human
**overrode it** to a free + bring-your-own-key model; the dialogue surfaced the
lever and let the owner make the call, where the one-shot simply shipped tiers
nobody endorsed.

### 2. CPA caught real blockers the one-shot missed

In round 1, the one-shot asserted a "no-Docker, bundled-backend" architecture
without ever checking its dependencies' licenses. Co-Architect's built-in audit
surfaced exactly that question, and verification found the backend's datastore is
under a Business Source License — then confirmed the specific use was permitted
and cleared it. A one-shot ships the unexamined assumption; the dialogue examined
it before it became a problem.

### 3. Engagement and ownership

Across both rounds the human *changed* recommendations (an inference-model choice,
a pricing model), *amended* the architecture, *challenged* the opening decision
(and was right to), and in round 2 drove the entire back half of the design solo
from a phone. One participant self-reported getting "wrapped up in the building."
A one-shot document is read once and filed; CPA produced decisions its owner
actually holds.

### 4. The honest cost

Co-Architect trades **session length and within-session interaction** for decision
ownership, error-catching, and a cheap, flat resume cost. For a quick task you'll
never revisit, a one-shot is faster and cheaper. For a consequential design you'll
come back to across days and devices, CPA's flat resume cost and owned decisions
are the win.

---

## What this is *not*

- **Not conclusive.** n = 1 per round. Two rounds that agree are encouraging, not
  proof.
- **Not a within-session token win.** CPA generates more conversational tokens
  than a single dump. Its wins are elsewhere (resume cost, ownership, caught
  errors) and we don't claim otherwise.
- **Not fully precise.** Durable-file token figures are char/4 approximations and
  the multi-device Arm B total wasn't reduced to one exact number. The control's
  56,622 is harness-measured and trustworthy; the rest are honest estimates.

## One-line takeaway

> Co-Architect doesn't win on raw generation tokens — it wins on **owned
> decisions, caught errors, and a small flat-cost resume artifact** that a growing
> transcript can't match. Across two rounds, the one-shot kept inventing the names
> and the pricing nobody asked for; the dialogue kept handing those calls back to
> the person who should make them.

---

### A note on one bug we found (and fixed)

Honesty cuts both ways. In *both* rounds, the mechanism that crosses devices — a
git-tracked living file — initially failed the same way: a mobile/cloud Claude
Code session silently committed the living file to an auto-created working branch
instead of the shared one the resume flow reads from. The work didn't disappear
(a fast-forward recovered it), but it meant the phone-pickup needed manual
cleanup. The skill now carries an explicit **branch guard** in its continuity
rules to prevent it. The dialogue format is what made the failure visible enough
to fix.
