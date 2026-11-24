# Gemini Prompt 03 — Project Audit (reusable)

Replace `<PROJECT>` and `<REPO>` before pasting.

```
Following AGENTS.md and GEMINI.md in Roids01/ai_ops, do a full-repo audit of <REPO> for project <PROJECT>.

Write the result to plans/YYYY-MM-DD-<PROJECT>-audit.md in ai_ops (status: draft, executor: claude).

Cover, ranked by severity:
1. Architecture: module boundaries, source vs. build output separation, where state lives.
2. Correctness and math: simulation or game-logic formulas, units, edge cases.
3. Performance: hot paths, and what will struggle on a Raspberry Pi 5.
4. Testability: what has no tests, and the 3 highest-value tests to add first.
5. Context hygiene: large assets or JSON blobs that agents should never load whole.

For each finding: file path(s), what's wrong, the concrete fix, and an acceptance check Claude Code can verify.
End with a recommended order of work (max 5 steps) and a proposed STATE.md row.
Do not modify the project repo.
```
