# Contributing

Contributions welcome! Here's how:

## Curricula
The easiest way to contribute is by adding new curriculum tracks. Copy `skills/sage-instructor/curricula/TEMPLATE.md`, fill it in, and submit a PR.

## Bug Reports
Open an issue describing: what you expected, what happened, and your Claude Code version.

## Plugin Development
1. Clone the repo
2. `claude plugin validate .` to verify structure
3. Test locally: install from the local directory
4. If you're editing `SKILL.md` or a curriculum's spec-level behavior (progress
   fields, verification, streaks, onboarding), run the relevant scenario(s) in
   `tests/scenarios/` — see `tests/README.md`. These are fixed regression
   scripts targeting bugs that were previously found and fixed; a spec edit
   that silently breaks one of them is exactly what they're for catching.
   `python3 tests/test_check_progress_schema.py` (also run in CI) covers the
   deterministic Tier 1 checks and needs no agent.
5. Bumping the plugin version? Run `tests/README.md`'s full pre-release
   checklist (all five Tier 2 scenarios) first.
6. Submit a PR with a clear description

## Code of Conduct
Be kind. Be constructive. We're all here to learn.
