---
name: recursive-loop-audit
description: Audit a named product, workflow, or codebase to find where a human is doing repeated manual diagnosis that an autonomous loop (sensor → policy → tool → quality gate → learning) could take over instead. Produces an evidence-based gap analysis across those five layers plus legibility, ephemeral-software, and human/AI-boundary checks, ending in a prioritized fix list. Trigger on "run a loop audit," "apply the recursive loop framework," Roman-legion/copilot-vs-loop references, "is this AI-native / self-improving," or "where could this be automated end-to-end." Different from tech-debt review (what's poorly built) or an ADR (which design/tech to pick) — this is about automation maturity of a recurring process, not code quality or a build-vs-buy call.
---

# Recursive Loop Audit

## What this is for

Most functions — in a company or in a codebase — run on a simple loop: something goes wrong, a person notices, investigates, and fixes it — and re-runs that same diagnosis every time it happens again. The alternative is a **recursive self-improving loop**: a system that senses what's happening, decides within defined limits what it's allowed to do about it, acts through real tools, checks its own work, and learns from the outcome — closing the cycle without a person re-doing the investigation each time.

This audit's job is to find, for one *named* target, exactly which of those layers already exist, which are missing, and where the gap actually is — grounded in real evidence (code, logs, tickets, docs), not a generic "AI could help here" essay. Anyone can assert that automation would help; the value of this skill is finding the *specific*, evidenced gap.

This context is for you, not the report. Don't re-explain the framework to the user — just apply it and show findings.

**A loop is a chain, not a scorecard.** All five layers have to actually exist and function for the target to have a real closed loop — not "4 out of 5 is pretty good." One missing or broken layer means there is no loop today, full stop, no matter how strong the other four are. Never let a report read as an overall positive impression when any layer is genuinely absent; state the verdict as blocked by that specific layer. See "Loop status" below for how this gets stated explicitly in every report.

This same all-steps-complete standard applies one level down, to this skill's own actions when it implements a quick win: a step that was skipped, unverifiable, or merely asserted is an open loop, not a closed one, no matter how confident the write-up sounds. See §Executing a quick win below for how that gets tracked in practice, state by state. Don't conflate the two: a single fix being fully Shipped/Verified/Confirmed does not mean the target's overall loop is closed — it means one layer took a step forward. Say both things separately.

## Before you start: pin the target

If the user has already named a concrete target in the conversation (a product, a repo, a workflow they've described), use it — don't ask again. If it's vague ("audit my company"), ask them to narrow it to one function or surface; auditing everything at once produces shallow, generic output for each part instead of a sharp read on one.

Before asking, check whether `.loop-audit/` already exists in this project. If it does, list the existing scopes and their targets (e.g. "search-flow → map-page, geo-search-api") so the user can continue one of those instead of starting fresh or accidentally creating a near-duplicate under a slightly different name. Always offer "define a new scope" too. Confirm this explicitly even when only one scope/target exists — don't silently assume it's the intended one just because it's the only option.

Then find out what evidence is actually reachable:
- A codebase or repo → read it. Don't summarize what a typical system in this domain looks like; read the actual files.
- A workflow with no code (support process, sales motion, ops cadence) → ask for whatever artifacts exist: a real ticket, a Slack thread, a doc, a walkthrough. If nothing is available, say so plainly in the output rather than inventing what a "typical" version of this process looks like.

If part of the target is genuinely inaccessible (a private tool you can't see, a process that only exists in someone's head), mark that layer's evidence as "not observed — ask [person/system]" instead of guessing. A confident-sounding guess is worse than an honest gap.

## Tracking progress across sessions

The "current state" section below reuses the exact template from **Output** — one report format, whether it's the chat response or the saved file.

Each audit is saved to `.loop-audit/<scope>/<target>.md` inside the target's own project — not in this skill's folder. Slugify scope and target names consistently (e.g. "Search Flow" and "search flow" must resolve to the same file), so repeat audits land on the same target instead of fragmenting into near-duplicates.

If the chosen target already has a file, read it in full before starting — not just the latest numbers, the whole prior report — and open the new report with what's changed across every layer since last time, instead of starting cold.

Before writing anything new, scan History for entries whose "check after" date has already passed. For each one, verify it first — pull the actual sensor data for that specific change — and log what you found (worked / partially / not yet / needs adjustment) as part of this run's findings. This is §Executing a quick win's step 8 (Validate at the interval), applied retroactively to whatever was left pending from last time. "What changed since last audit" should be built from that verification when there's something due, not written as a generic diff.

The file has two parts:
- **Current state** — the full findings (all five layers, the three checks, the prioritized list) as they stand right now. This gets replaced each run; it should never just keep growing.
- **History**, at the bottom — one short line per run, not a full report copy. When an entry has a delayed effect, include when to check back: `2026-08-15: sensor layer fixed (analas quota resolved) — needs ~14 days of usage data, check after 2026-08-29`. Also log human-touch points — moments a human had to approve, decide, or manually verify something, and why. Across repeated runs this becomes real evidence for the Policy layer: whichever touch point recurs most is the next best candidate to automate. If this passes ~20 entries, trim the oldest and note "earlier entries trimmed" rather than letting the file grow indefinitely.

## The five layers

Work through each one and write down, with evidence, what exists and what's missing. Skip layers that genuinely don't apply to this target rather than padding — a simple internal tool might have no meaningful "policy layer" yet, and that's a real finding, not an omission to cover up.

1. **Sensor layer** — What raw signal exists about how this thing is actually performing in the real world? Logs, error events, support tickets, usage analytics, user behavior, code changes, cancellations. Critically: is that signal *actually flowing right now*, or silently broken (wrong config, a vendor quota exceeded, an untracked event)? A sensor that used to work but is currently dead is a more urgent finding than one that was never built.

   Some sensor always exists, even if it's just a person noticing by eye — the real question is whether it's strong enough to confirm a fix actually worked. If sensor sources aren't obvious from the evidence already gathered, ask the user directly with concrete offers rather than a vague question: error/exception tracking, product analytics, support tickets, session recordings, logs. If what exists is inadequate — missing entirely, or present but noisy/too coarse/measuring the wrong thing — say so explicitly and name a specific alternative or addition (e.g. "X is used today but only shows Y; Z would confirm this specific kind of fix"), not just "consider adding measurement." Ask the user if they want it added.

   A missing or weak sensor deserves special weight in "what to address first" — without it, nothing else this audit recommends can ever be confirmed as working or not.

   When proposing a sensor or a fix, state how long until its effect is checkable — instant (check right after) or delayed (needs some number of days of real usage) — and note the date the change was made, so a later check knows whether enough time has passed to trust the signal.
2. **Policy / decision layer** — What is the system (or team) currently allowed to do without asking a human first, versus what requires sign-off or must be logged? If the honest answer is "nothing is automated, a human approves every single change," that itself is the finding — it means there's no policy layer yet, only a person acting as the entire policy.

   When proposing what a policy layer should look like, push for the narrowest human-touch boundary that's still safe — name specific categories that could safely auto-proceed (small, reversible, covered by tests/evals) versus ones that genuinely need a human. The goal is *minimum necessary* touch, not zero: requiring a human to approve a one-line typo fix over-touches just as much as letting a pricing change ship alone under-touches. (This is guidance for evaluating or designing the *target's* policy layer. This skill's own quick-win fixes are governed by a separate, stricter rule regardless of size — see §Executing a quick win — so don't read the target's auto-proceed criteria as license for this skill to skip planning its own fixes with the user.)
3. **Tool layer** — What deterministic, callable actions actually exist that could act on a diagnosis? An API to query, a script, a config file that could be edited and merged, a deploy pipeline.

   If tools aren't obvious from the evidence, ask the user directly with concrete offers rather than "what can act on this" — a CI/CD or deploy pipeline, an internal API, a scripting/automation tool, the ability to open a PR programmatically. Keep "exists and callable today" separate from "would need to be built" — that distinction is what determines how big a lift automating any given fix actually is.
4. **Quality gate** — What stands between "a fix was proposed" and "it shipped"? Tests, evals, staging checks, human review, safety filters. No test suite / no CI / no review step at all is a real, load-bearing finding — it caps how much of the rest of this loop could safely run without a human, no matter how good the other layers get.

   Before concluding a quality gate is missing, check for it directly rather than assuming: CI config files (`.github/workflows`, `.gitlab-ci.yml`, `Jenkinsfile`, `azure-pipelines.yml`), a test runner config or test script (`package.json` test script, `jest`/`vitest`/`pytest` config, `*_test.go` files), test files themselves (`*.test.*`, `*.spec.*`, a `tests/` or `__tests__/` folder), pre-commit/pre-push hooks, branch protection rules, an existing eval suite. Note exactly which of these exist and which don't — absence of one doesn't mean absence of all.

   When recommending what to add, give one complete, concrete proposal for this project — not "add tests" in the abstract, but tailored to what's already there: e.g. "you have a lint step in CI but no test runner; add X, wired into the existing pipeline at Y, triggered on Z." This is what makes the policy layer's "auto-proceed" categories actually safe — without it, no amount of trust in the other layers makes autonomy safe to grant.

   This layer applies reflexively too: when this skill implements a quick win itself, the same standard governs whether that specific change can be called verified. See §Executing a quick win.
5. **Learning mechanism** — Is there anything that actually closes the loop today — observing a failure, diagnosing it, shipping a fix, and confirming it worked — without a person re-running that whole investigation by hand next time the same thing happens? Look for a real precedent in the evidence: a past incident, a hardcoded workaround, a comment explaining "we found this the hard way." That's usually the clearest sign of what the learning mechanism *should* automate, because it already happened once manually.

## The three cross-cutting checks

These apply on top of the five layers, not instead of them:

- **Legibility** — Is the domain knowledge this loop would depend on consolidated somewhere an agent (or a new hire) could actually read and act on, or is it scattered — duplicated across files, tribal knowledge in someone's head, buried in chat history? A loop can't operate on knowledge it can't see. This especially blocks Layer 5 (Learning mechanism) — an autonomous loop can't safely close itself on knowledge it can't find or read.
- **Ephemeral software vs. durable context** — Is the current implementation a one-off hardcoded guess, or expressed as config/rules that could be regenerated or swapped out as understanding improves? The distinction matters because it determines whether future improvement means editing code by hand forever, or updating a rule that regenerates the implementation.
- **Human/AI boundary** — Which decisions here are genuinely high-stakes judgment calls that should stay human — not because automation hasn't gotten there yet, but because the cost of a wrong autonomous call is too high (irreversible, emotionally sensitive, legally consequential, or dependent on context no sensor captures)? Name these explicitly. A loop audit that recommends automating everything is missing something real; the point is to find where automation helps *and* where it deliberately shouldn't reach.

## Loop status (target-level verdict)

After the layer-by-layer breakdown, state an explicit, unambiguous verdict on whether the *target* has an actual working loop right now — not a quality impression, a binary chain check. A loop closes only when all five layers are present, functioning, and actually connected to each other (sensor feeds the decision, the decision drives the tool, the tool's output passes the gate, the outcome feeds learning, learning improves the sensor/policy next time). Evaluate it as a chain:

- **No loop** — two or more layers are missing/broken, or the one existing "loop" is just a person doing the whole cycle by hand (which is what layers 2-5 replace, not a loop itself).
- **Partial loop, blocked by [layer]** — most layers exist and work, but one specific layer is missing, broken, or disconnected from the rest. Name that layer specifically; it's the one thing standing between this target and a real loop, and it's what "what to address first" should be organized around.
- **Closed loop** — all five layers are present, functioning, and connected end-to-end for this specific target, right now, with evidence for each link, not just each node.

Do not average or round up. A target with a great sensor, a great tool, and a great quality gate but *no* learning mechanism is "No loop" / "Partial loop, blocked by Learning" — not "4/5, mostly there." The whole point of the chain framing is that a strong sensor nobody acts on, or a fix nobody re-verifies, produces zero autonomous value even if every other layer is excellent.

A quick win implemented during the audit advances at most one layer. It never by itself changes the target's Loop status verdict from the prior run unless that layer was *the* blocking one and every other layer already existed — say this explicitly rather than letting a "Quick win — Shipped, Verified" item read as if the target now has a closed loop.

## Executing a quick win

A **quick win** is a fix small and reversible enough that, once its plan is confirmed, the *implementation* doesn't need a second round of permission — as distinct from a **Structural bet**, which only ever gets proposed and scoped in this audit, never built here. Being small and reversible earns a fix that classification; it does not earn it a skip of the confirmation step below. Every quick win this skill acts on follows this sequence, in order, without skipping or reordering steps:

1. **Measurement ready** — decide the real-world signal that will show whether the fix actually did what was intended: the Sensor-layer check this fix will ultimately be judged by. Confirm that signal is actually collectible in the current environment. If nothing adequate exists yet, say so now rather than discovering it later.
2. **Quality gates ready** — identify the executable check (build, typecheck, lint, test) that will confirm the fix landed correctly. If the target's existing gates don't cover the code being touched, add the missing test/check as part of this step, before writing the actual fix — don't leave "we should add a test for this" as a dangling note in the output.
3. **Discuss and confirm with the user** — present both the measurement plan (step 1) and the quality-gate plan (step 2) together, and get the user's explicit go-ahead before implementing anything. This step applies to every quick win, regardless of size — small-and-reversible is what makes the *implementation* autonomous once confirmed, not a license to skip planning it with the user first.
4. **Implement** — make the change (delegate to a subagent if that fits the task).
5. **Run the quality gate** — execute the step-2 check now that the change exists. If it fails, fix and re-run before moving on; a failing gate blocks the next step, it doesn't get waived.
6. **Deploy and run** — land the change where it actually operates. If this means publishing or pushing somewhere visible to others, that still needs its own pause-and-ask per this project's general action rules — confirming the plan in step 3 is not the same as confirming the push.
7. **Start measurement** — begin collecting the step-1 signal, for whatever interval was established there (instant, or delayed and needing real usage). Record the "check after" date in the target's History log (see §Tracking progress across sessions).
8. **Validate at the interval** — once the interval passes, pull the actual signal and state plainly whether the change did what was intended, partially, or not at all.

### Reporting where a fix actually stands

Track these states separately — never collapse them into one "done":

- **Planned** — steps 1–2 are decided but step 3 (user confirmation) hasn't happened yet. This is where every quick win starts in the audit report itself (see §Output) — a plan awaiting go-ahead, not a change already made.
- **Shipped** — steps 3–4 happened: the user confirmed, and the change was made. Alone, this is never enough to call something "done."
- **Verified** — step 5 ran and passed: an actual executable check (build, typecheck, lint, test, or a manually-triggered request that returned the expected response), not just self-review. If a check exists and is runnable, run it — don't skip straight to self-review. If no check exists, or the environment blocks the one that does (unreachable registry, missing credentials, no test to run), self-review by hand is the fallback — but it must be labeled as a fallback ("reviewed by hand, no automated gate available/runnable"), never phrased as equivalent to a passing check.
- **Confirmed** — step 8 happened: real-world sensor data shows the fix is actually doing its job, not just that it shipped without error.

A single fix is a **closed loop for that action** only when every state that applies to it is true, through Confirmed. Name exactly where it stopped — "Planned, awaiting your go-ahead," "Shipped, unverified," "Shipped, Verified, confirmation pending until [date]," or "Shipped, Verified, Confirmed — [worked / partially / not yet / needs adjustment]" — never a flat "done," and never a state further along than what actually happened. This is a narrower claim than §Loop status above: it's about one action, not the target's overall automation maturity — a fix reaching Shipped/Verified/Confirmed does not mean the target now has a closed loop, it means one layer took a step forward. Say both things separately.

## Output

Always structure the response this way, in order. Keep every claim tied to something concrete — a file path, a quoted log line, a specific ticket, or an explicit "not observed, needs input from X" — never a generic industry-best-practice statement standing in for evidence about this target.

```markdown
# Loop audit: [scope] → [target name]

## What this is (one line)
[what the target does / who it serves]

## What changed since last audit
[only if `.loop-audit/<scope>/<target>.md` already existed — otherwise omit this section entirely]

## Layer-by-layer

### Sensor layer
- What exists: ...
- What's missing / broken: ...

### Policy layer
- What exists: ...
- What's missing: ...

### Tool layer
- What exists: ...
- What's missing: ...

### Quality gate
- What exists: ...
- What's missing: ...

### Learning mechanism
- What exists: ...
- What's missing: ...
- Precedent found (if any): [a past manual fix that shows what this should have automated]

## Loop status
[No loop / Partial loop, blocked by [layer] / Closed loop — per §Loop status, evaluated as a chain, not averaged. Name the specific blocking layer(s) if not closed.]

## Cross-cutting checks
- Legibility: ...
- Ephemeral software vs. durable context: ...
- Human/AI boundary — what should stay manual and why: ...

## What to address first
Ranked list, each tagged as **Quick win** (small, immediate) or **Structural bet** (bigger, needs its own scoping), each tied back to a specific layer/check above — not a fresh list of generic suggestions. For each quick win, state the measurement plan and quality-gate plan (§Executing a quick win, steps 1–2) and mark it **Planned — awaiting your go-ahead**; do not implement, deploy, or claim any further state in the same turn the audit is first delivered. If the user confirms during this conversation, proceed through the remaining steps and update the status using the vocabulary from §Executing a quick win, naming exactly which step was reached. Never write "done" or "fixed" for a quick win that stopped at Planned. And never let a fully Shipped/Verified/Confirmed quick win imply the target's overall Loop status changed unless it explicitly did — most quick wins nudge one layer forward without closing the loop.
```

If the target has no code/data to inspect at all (pure conceptual discussion), say so up front and produce the best-effort version anyway, clearly marking every layer as based on the user's description rather than direct evidence — don't silently pretend the rigor is the same as when real artifacts were reachable.

After delivering the report, if any finding has a "check after" date, offer to schedule the next audit run for that date — otherwise it's just a note nobody acts on.

## What to avoid

- Don't produce a "here are 5 AI best practices" essay — every section should be identifiably about *this* target, not swappable with any other audit.
- Don't force all 5 layers to have equally rich findings — some genuinely won't apply yet, and saying so is more useful than padding.
- Don't recommend automating a human/AI-boundary decision just to fill out the "what to address first" list — the boundary section exists precisely to protect against that.
- Don't restate the target's own marketing description as the "what this is" line — describe it functionally, from what the evidence actually shows it does.
- Don't implement a quick win before the user has confirmed the measurement and quality-gate plan (§Executing a quick win, step 3) — even for a change that looks trivially small and reversible.

## This skill should improve too

The same question this skill asks of its targets applies to itself: is running this audit a one-off manual cycle, or does it get better from real use? If a past recommendation turned out wrong, or the template didn't fit a target well, that's feedback worth folding back into this file — not something to silently work around each time. Revisit this skill periodically through deliberate iteration (test → feedback → refine), the same way it was built.

**Revision log:**
- 2026-07-21: A quick win was reported as "done" in the same output that also admitted its quality gate never ran (a build/typecheck blocked by an unreachable private registry). User feedback: "loop means all the steps should pass so we call it one loop" — self-review isn't the same as a passing gate, and a report shouldn't be able to say both "done" and "unverified" without contradicting itself. Added a "Reporting a fix's status" section (Shipped / Verified / Confirmed as separate, trackable states for one action) and threaded that vocabulary through the quality-gate layer, the auto-proceed instruction, and the Output template.
- 2026-07-21 (same day, follow-up): User clarified the first fix was too narrow — "its not only about that step all the steps should be done." The real gap was one level up: the skill graded each of the five layers independently and never forced an explicit verdict on whether the *target* has an actual closed loop end-to-end. A target could read as "mostly good" with 4 strong layers and 1 missing one, when the correct read is "no loop, blocked by that one layer" — a loop is a chain, not an average. Added a separate §Loop status section (target-level, chain logic: No loop / Partial loop blocked by [layer] / Closed loop) and a required "## Loop status" block in the Output template, distinct from the action-level fix-status tracking. Explicitly noted a quick win closing one layer does not imply the target's overall Loop status changed.
- 2026-07-21 (same day, second follow-up): A live run exposed a sequencing gap — a quick win (deduping a duplicated SVG icon map) was implemented first and verified after, and the verification step happened to catch a transcription bug the author introduced during the edit; separately, a browser-based "Confirmed" check turned out to be infeasible in the sandbox, discovered only after the fix had already shipped. User asked whether the skill should require the measurement/check to be ready *before* implementing, "something like TDD." Agreed this was a real gap but full TDD (test-first authorship) doesn't generalize to non-code targets (a workflow or config fix may have no test to write). Added a narrower principle: name the specific acceptance check and confirm it's actually runnable in the current environment *before* touching the change, not after — the TDD principle (decide the check first) without the TDD mechanic (write a failing test first).
- 2026-07-21 (same day, third follow-up): User specified the intended flow in full and explicitly, superseding the lighter touch of the second follow-up: measurement ready → quality gates ready (adding tests here if missing) → discuss and confirm with the user → implement → run quality gate → deploy and run → start measurement until the interval → validate at the interval. The key addition beyond the prior edit is an explicit, mandatory user confirmation of both the measurement plan and the quality-gate plan before implementation begins, for every quick win regardless of size. This changes what "auto-proceed" means: it now governs the implementation and deploy steps once the plan is confirmed, not the decision to build the plan and act on it unasked.
- 2026-07-21 (same day, full rewrite): User asked for a complete rewrite rather than another patch, since three same-day edits had left the file with overlapping sections — a "Reporting a fix's status" section and the newer 8-step sequence both defined Shipped/Verified/Confirmed, and the Output template's "What to address first" instructions still said quick wins are "reported as attempted, not proposed," directly contradicting the new step-3 mandatory-confirmation rule (a quick win can't be "attempted" before the user has confirmed it). Consolidated everything about executing and reporting a fix into one §Executing a quick win section (the 8 steps plus a four-state status vocabulary: Planned / Shipped / Verified / Confirmed, replacing the old three-state version so "awaiting confirmation" has its own named state instead of being unrepresentable). Rewrote the Output template's quick-win instructions so a first-delivered audit reports quick wins as **Planned**, not as already done. Added a clarifying note under the Policy layer distinguishing target policy-layer guidance (what a target *should* let auto-proceed) from this skill's own stricter, always-confirm-first rule for its own fixes, so the two don't get conflated. Added a "What to avoid" bullet naming the step-3 skip directly.
