# moshirv-tools

Claude Code plugin marketplace with two plugins:

- **recursive-loop-audit** – audit a product, workflow or codebase for manual diagnosis an autonomous loop could take over.
- **supervised-build** – plan a feature as a living `*_PLAN.md` with numbered steps, then build it step by step by handing each step to a cheaper agent (fresh, small context) while you supervise: write the brief, review the diff and tests, merge, update the plan.

## Install

```
claude plugin marketplace add MoShirv/recursive-loop-audit-plugin
claude plugin install supervised-build@moshirv-tools
claude plugin install recursive-loop-audit@moshirv-tools
```
