---
name: supervised-build
description: Plan a feature as a living plan file with numbered steps, then build it step by step by handing each step to a cheaper agent (fresh, small context) while you act as supervisor - write the brief, review the diff and tests, merge, update the plan, and give the user a manual test checklist and the next step. Use when the user wants to design and then build something larger than one PR, says "make a plan with steps", "give each part to a cheaper agent", "you supervise", "use sub agents", or asks to continue a plan file like CRM_PLAN.md / *_PLAN.md. Not for one-file fixes or pure research.
---

# Supervised Build

You are the **supervisor** (expensive model, holds the big picture). **Workers** are cheaper agents with
fresh, small contexts that each build exactly one step. The **plan file** is the shared memory between
you, the workers and the user - so nobody needs the conversation history.

Token rules (the reason this skill exists):
- Workers start **fresh** (never fork your conversation) on a **cheaper model** (Sonnet-class; Haiku only
  for mechanical edits). Their brief + the plan file is all the context they get.
- Point workers at files and plan sections; don't paste code or long history into briefs.
- You never re-implement a worker's step. You read `git diff --stat` first, then only the risky parts.
- Workers run only the tests they add/touch plus named regression files - never the full suite.
- One small step per worker. Start small; grow step size only after a step lands cleanly.

## Phase 1 - Design (with the user, in chat)
1. Explore only what the design needs (grep/read targeted files; check infra/docs repos if relevant).
2. Talk it through: requirements, options with a recommendation, the user's decisions. Ask only real
   decisions; give a default for everything else.
3. Write the plan file in the repo root (`<TOPIC>_PLAN.md`) from `templates/plan.md`. Must have:
   "Where we are" line, Decisions, numbered Steps tables with status, Open decisions tagged with the
   step they block, Infra tasks for the user.
4. Save a short project memory pointing at the plan file and the user's key decisions/preferences.

## Phase 2 - Build loop (one step at a time)
For each next unblocked step:
1. **Brief** - fill `templates/brief.md`: background (>=80 chars, stands alone), scope, data/behaviour,
   tests required, rules, "done means". Name exact files/patterns to copy. State what the *next* step
   needs from this one (e.g. a nullable key) so it isn't redone later.
2. **Start** the worker: fresh context, cheaper model, own worktree/branch cut from the latest main (or
   the previous step's branch if not merged yet), `wake_when_done`. Then end your turn - don't poll.
3. **Redirect** if the user changes scope mid-step: message the worker (by session id) with the change;
   early redirects are cheap. If it dies on a network/API error, message it to check `git log` and resume.
4. **Review** when it reports done:
   - `git log main..<branch>`, `git diff --stat main...<branch>`, then read new core code and any
     change to *existing* tests or code (each needs a reason; narrowed assertions are ok if intent stays).
   - Check the project rules: no changes to prod data or old migrations, no dead code/fields,
     no N+1, permissions, separate-DB boundaries, comment style.
   - Small fixes: message the worker. Never silently fix big things yourself.
5. **Ship** (only as the user authorised): push, PR with a short body (what, deploy notes, tests),
   merge. If the user hasn't authorised merging, stop at the PR. If a permission check blocks a merge,
   tell the user - never work around it.
6. **Report** to the user, short:
   - PR link + what changed.
   - One-time server commands, if any, as separate `bash` blocks (the user runs server/credential steps).
   - A **manual test checklist** for production (numbered, concrete clicks/inputs and expected result,
     incl. access per role and "nothing else broke"). Combine earlier steps' checks when asked.
   - **What's next**: the next unblocked step(s) and what's blocked on the user.
7. **Update** the plan (status, "Where we are") - the worker does it in its commit; you fix it if not.
   Batch doc-only edits into the next merge instead of separate commits.

## Rules that always apply
- The user's project rules (memory/CLAUDE.md) go into every brief's Rules section.
- Workers never push, merge, open PRs or touch servers. Supervisor never runs server commands - it hands
  them to the user.
- Run at most two workers at once, and only on steps that don't touch the same files.
- Steps waiting on a user decision stay ⏳; ask for the decision in the report, don't guess it.
- When the user defers something ("put it at the end"), move it in the plan and note why.
