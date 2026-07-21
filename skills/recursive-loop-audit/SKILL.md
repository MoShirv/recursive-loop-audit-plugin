---
name: recursive-loop-audit
description: Audit a named product, workflow, or codebase to find where a human is doing repeated manual diagnosis that an autonomous loop (sensor → policy → tool → quality gate → learning) could take over instead. Produces an evidence-based gap analysis across those five layers plus legibility, ephemeral-software, and human/AI-boundary checks, ending in a prioritized fix list. Trigger on "run a loop audit," "apply the recursive loop framework," Roman-legion/copilot-vs-loop references, "is this AI-native / self-improving," or "where could this be automated end-to-end." Different from tech-debt review (what's poorly built) or an ADR (which design/tech to pick) — this is about automation maturity of a recurring process, not code quality or a build-vs-buy call.
---

# Recursive Loop Audit

## What this is for

Most functions — in a company or in a codebase — run on a simple loop: something goes wrong, a person notices, investigates, and fixes it — and re-runs that same diagnosis every time it happens again. The alternative is a **recursive self-improving loop**: a system that senses what's happening, decides within defined limits what it's allowed to do about it, acts through real tools, checks its own work, and learns from the outcome — closing the cycle without a person re-doing the investigation each time.

This audit's job is to find, for one *named* target, exactly which of those layers already exist, which are missing, and where the gap actually is — grounded in real evidence (code, logs, tickets, docs), not a generic "AI could help here" essay. Anyone can assert that automation would help; the value of this skill is finding the *specific*, evidenced gap.

This context is for you, not the report. Don't re-explain the framework to the user — just apply it and show findings.

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

Before writing anything new, scan History for entries whose "check after" date has already passed. For each one, verify it first — pull the actual sensor data for that specific change — and log what you found (worked / partially / not yet / needs adjustment) as part of this run's findings. "What changed since last audit" should be built from that verification when there's something due, not written as a generic diff.

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

   When proposing what a policy layer should look like, push for the narrowest human-touch boundary that's still safe — name specific categories that could safely auto-proceed (small, reversible, covered by tests/evals) versus ones that genuinely need a human. The goal is *minimum necessary* touch, not zero: requiring a human to approve a one-line typo fix over-touches just as much as letting a pricing change ship alone under-touches.
3. **Tool layer** — What deterministic, callable actions actually exist that could act on a diagnosis? An API to query, a script, a config file that could be edited and merged, a deploy pipeline.

   If tools aren't obvious from the evidence, ask the user directly with concrete offers rather than "what can act on this" — a CI/CD or deploy pipeline, an internal API, a scripting/automation tool, the ability to open a PR programmatically. Keep "exists and callable today" separate from "would need to be built" — that distinction is what determines how big a lift automating any given fix actually is.
4. **Quality gate** — What stands between "a fix was proposed" and "it shipped"? Tests, evals, staging checks, human review, safety filters. No test suite / no CI / no review step at all is a real, load-bearing finding — it caps how much of the rest of this loop could safely run without a human, no matter how good the other layers get.

   Before concluding a quality gate is missing, check for it directly rather than assuming: CI config files (`.github/workflows`, `.gitlab-ci.yml`, `Jenkinsfile`, `azure-pipelines.yml`), a test runner config or test script (`package.json` test script, `jest`/`vitest`/`pytest` config, `*_test.go` files), test files themselves (`*.test.*`, `*.spec.*`, a `tests/` or `__tests__/` folder), pre-commit/pre-push hooks, branch protection rules, an existing eval suite. Note exactly which of these exist and which don't — absence of one doesn't mean absence of all.

   When recommending what to add, give one complete, concrete proposal for this project — not "add tests" in the abstract, but tailored to what's already there: e.g. "you have a lint step in CI but no test runner; add X, wired into the existing pipeline at Y, triggered on Z." This is what makes the policy layer's "auto-proceed" categories actually safe — without it, no amount of trust in the other layers makes autonomy safe to grant.
5. **Learning mechanism** — Is there anything that actually closes the loop today — observing a failure, diagnosing it, shipping a fix, and confirming it worked — without a person re-running that whole investigation by hand next time the same thing happens? Look for a real precedent in the evidence: a past incident, a hardcoded workaround, a comment explaining "we found this the hard way." That's usually the clearest sign of what the learning mechanism *should* automate, because it already happened once manually.

## The three cross-cutting checks

These apply on top of the five layers, not instead of them:

- **Legibility** — Is the domain knowledge this loop would depend on consolidated somewhere an agent (or a new hire) could actually read and act on, or is it scattered — duplicated across files, tribal knowledge in someone's head, buried in chat history? A loop can't operate on knowledge it can't see. This especially blocks Layer 5 (Learning mechanism) — an autonomous loop can't safely close itself on knowledge it can't find or read.
- **Ephemeral software vs. durable context** — Is the current implementation a one-off hardcoded guess, or expressed as config/rules that could be regenerated or swapped out as understanding improves? The distinction matters because it determines whether future improvement means editing code by hand forever, or updating a rule that regenerates the implementation.
- **Human/AI boundary** — Which decisions here are genuinely high-stakes judgment calls that should stay human — not because automation hasn't gotten there yet, but because the cost of a wrong autonomous call is too high (irreversible, emotionally sensitive, legally consequential, or dependent on context no sensor captures)? Name these explicitly. A loop audit that recommends automating everything is missing something real; the point is to find where automation helps *and* where it deliberately shouldn't reach.

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

## Cross-cutting checks
- Legibility: ...
- Ephemeral software vs. durable context: ...
- Human/AI boundary — what should stay manual and why: ...

## What to address first
Ranked list, each tagged as **Quick win** (small, immediate) or **Structural bet** (bigger, needs its own scoping), each tied back to a specific layer/check above — not a fresh list of generic suggestions.
```

If the target has no code/data to inspect at all (pure conceptual discussion), say so up front and produce the best-effort version anyway, clearly marking every layer as based on the user's description rather than direct evidence — don't silently pretend the rigor is the same as when real artifacts were reachable.

For any Quick win small and reversible enough to qualify as auto-proceed under the Policy layer's own rule, don't just describe it in the report — offer to make the change right now (draft the edit, open the PR), instead of leaving it as a suggestion for someone to implement later.

After delivering the report, if any finding has a "check after" date, offer to schedule the next audit run for that date — otherwise it's just a note nobody acts on.

## What to avoid

- Don't produce a "here are 5 AI best practices" essay — every section should be identifiably about *this* target, not swappable with any other audit.
- Don't force all 5 layers to have equally rich findings — some genuinely won't apply yet, and saying so is more useful than padding.
- Don't recommend automating a human/AI-boundary decision just to fill out the "what to address first" list — the boundary section exists precisely to protect against that.
- Don't restate the target's own marketing description as the "what this is" line — describe it functionally, from what the evidence actually shows it does.

## This skill should improve too

The same question this skill asks of its targets applies to itself: is running this audit a one-off manual cycle, or does it get better from real use? If a past recommendation turned out wrong, or the template didn't fit a target well, that's feedback worth folding back into this file — not something to silently work around each time. Revisit this skill periodically through deliberate iteration (test → feedback → refine), the same way it was built.
