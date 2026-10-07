<!-- Worker brief. `background` and `prompt` must each stand alone: the worker has NOT seen the conversation. -->

## background (>= 80 chars)
<Project in one line (stack, repo kind).> We are building <feature>, planned in <PLAN FILE> — read
"<sections>" first. <Architecture constraints the worker must respect, e.g. separate DB, queues,
naming.> Already built: <previous steps, with file paths to copy patterns from>. The NEXT step after
yours is <x>, so <what to leave room for>. A supervisor session reviews your diff.

## prompt
Implement <PLAN FILE> step <n>: <one-line goal>.

### Scope
1. <Data: tables/columns, each one used>
2. <Behaviour: exact rules, edge cases>
3. <UI/API: where, who can see/do what>

### Tests (complete coverage is required)
- <behaviours, edge cases, access matrix, no-N+1 / bounded queries, nothing else changed>
- Run only the tests you add/touch plus these regression files: <paths>, with `<test command> --parallel`,
  until green. Not the whole suite.

### Rules
- <project rules from memory/CLAUDE.md: e.g. never modify existing data or migrations, no dead code>
- Match surrounding style and comment density.
- Do NOT push, merge, open PRs or touch any server. ONE commit on your branch ending with:
  Co-Authored-By: <worker model> <noreply@anthropic.com>
- In <PLAN FILE> mark step <n> ✅ (with any one-time command) and update "Where we are".

### Done means
One commit; tests green; final report: files changed, choices made and why, anything left out,
test command + result per file, anything unsure.
