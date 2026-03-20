# Contributing to skill-cooker

Thank you for your interest in contributing! This document explains how to get involved.

---

## Types of contributions welcome

- **Bug reports** — skill-cooker behaves unexpectedly during execution
- **Skill quality feedback** — the generated skill doesn't meet expectations
- **Feature requests** — new capabilities or workflow improvements
- **Documentation improvements** — clearer explanations, better examples
- **Translations** — additional language versions of SKILL.md

---

## Reporting issues

Before opening an issue, please check whether a similar one already exists.

When filing a bug report, include:
- Which mode you were running (Full Mode / Degraded Mode)
- The natural-language requirement you provided
- Which step the unexpected behavior occurred at
- What you expected vs. what actually happened

Use the appropriate issue template in `.github/ISSUE_TEMPLATE/`.

---

## Submitting a pull request

1. Fork the repository and create a branch from `main`
2. Make your changes
3. Ensure SKILL.md stays under 500 lines (move detail to `references/` if needed)
4. If you're modifying role context boundaries, update `references/roles.md` to match
5. If you're modifying convergence logic, update `references/convergence.md` to match
6. Open a PR with a clear description of what changed and why

---

## Skill quality standards

Any change to SKILL.md or its reference files should be validated against these criteria:

- All role context boundaries remain explicitly declared (constraint 9)
- Convergence conditions remain dual-AND (constraint 4)
- Both human confirmation checkpoints are preserved
- Degraded Mode declarations are complete and honest about limitations
- No step silently degrades without user notification

If you're adding new behavior, include at least one new assertion in `evals/evals.json`
that covers the new behavior.

---

## SKILL.md writing style

- Use imperative mood for instructions
- Explain *why* behind constraints, not just *what*
- Mark Degraded Mode variants with the `🔸` prefix for visual consistency
- Avoid MUST/NEVER in all-caps where possible — explain the reasoning instead
- Keep the main SKILL.md focused on flow; move detailed reference material to `references/`

---

## Questions?

Open a Discussion on GitHub if you're unsure whether something is a bug or intended behavior.
