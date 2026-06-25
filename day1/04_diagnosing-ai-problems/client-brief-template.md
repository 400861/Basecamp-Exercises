# Client Brief — Meridian Support Pilot

*One page. Fill every line. This is what Priya takes to her leadership.*

---

**Client & pilot**
Meridian — an AI agent that triages and resolves customer support tickets (a coordinator routing to billing / technical / account specialists). Live 3 weeks; closing tickets that aren't actually resolved.

**What's actually breaking it**
*(Name it plainly. Not the model — the system around it.)*

Not the model — the coordinator's instructions. Two flaws: (1) it routed each ticket to a single specialist, so any ticket raising more than one problem got only one of them handled (T-4471 raised an SSO/access issue *and* a billing issue; the agent fixed one and dropped the other — the "misses things a junior rep would catch"). (2) It defaulted to closing tickets as "resolved" regardless of whether the work was actually done — which is how a customer was told their issue was fixed when it wasn't.

**The fix**
*(What we changed, and where. Which prompt, which tool, which line.)*

Rewrote `system-prompt-coordinator.txt`. Step 5 (DELEGATE) now spawns one specialist per *distinct* issue, not one per ticket. Step 6 (SYNTHESIZE AND CLOSE HONESTLY) now sets resolution status to match what was actually actioned — "resolved" only if every issue was handled, "escalated" when no specialist or tool can perform what's asked (e.g. a human-only authorization), and never claims an action the system didn't perform. Also de-collapsed the classification guide, gave an explicit canonical tool list, and added a multi-issue worked example. No model change, no code change to the scoring/graders.

**Proof**
*(Before → after. Quality score moved from ____ to ____. Cost held at / dropped to $____ per run.)*

T-4471 RESOLVED rate: **0/5 → 5/5** (and holds at 5/5 on a repeat run). Held-out tickets it had never seen (different customers, different problems): **6/9 → 9/9** (15/15 in full validation) — so the fix generalizes, it isn't tuned to one ticket. Cost held steady at **~$0.16 per trial**. Same model throughout (`claude-sonnet-4-6`).

**What it would take**
*(Rough scope, and the constraint to hit — e.g. stay within current per-ticket cost.)*

The change is already made and validated against the bad ticket and the held-out set. Remaining scope: a light regression pass over a broader ticket sample before re-enabling auto-close in production. Constraint held: stay within the current ~$0.16/ticket cost.

**The objection we'll get**
*("Why not just use a better model?" — answer it with the numbers above.)*

We tested it directly. The same ticket on the larger, premium model (`claude-opus-4-8`) also resolved 5/5 — but at **$0.33/trial, roughly 2×**, for zero improvement over the fix we already shipped. And a bigger model would *not* have fixed the false-"resolved" bug, because that was an instruction defect, not a capability gap. Upgrading the model would have cost more every day and fixed nothing.
