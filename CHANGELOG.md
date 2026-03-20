# Changelog

All notable changes to skill-cooker are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [0.1.0] — 2026-03-20

### Added

- Initial release of skill-cooker
- Two-draft architecture: tmp_claude (intuitive draft) + tmp_standard (spec-compliant draft)
- Three-role cognitive isolation model:
  - Role I: Pure-Requirement Generator (no skill-creator exposure)
  - Role II: Standard Builder (no tmp_claude anchoring)
  - Role III: Independent Evaluator (never generates skill content)
- Full Mode (Claude Code): genuine blank context windows via subagents
- Degraded Mode (Claude.ai): forced-forgetting declarations as approximation
- Dynamic eval design: assertions based on requirement spec, not implementation
- Dual convergence condition: pass rate ≥ 85% AND two consecutive rounds improvement < 5%
- Local optimum detection and user notification
- Hard cap of 5 iterations with clear user decision options
- Two human confirmation checkpoints (per-round + final)
- Atomic file operations with backup-before-delete ordering
- Assertion set lock rule: reset round counter if assertions change mid-iteration
- Statistical validity guard: minimum 5 assertions before computing pass rates
- Structured diff presentation: removed content listed separately for user confirmation
- Parallel execution support for stages 3-A and 3-B in Claude Code
- English translation of all SKILL.md and reference files
- 9 explicit design constraints documented and enforced
- Reference files: roles.md, eval-schema.md, convergence.md
- MIT License
