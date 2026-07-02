# Changelog

## [Unreleased]

### Added
- **`tests/` regression harness.** Closes the gap where every fix in 1.2.0
  was validated by a one-off simulated session with no way to re-check it
  after a future edit. Two tiers:
  - `tests/check_progress_schema.py` — deterministic, no-LLM structural
    checker for `.sage-progress.json`/`.sage-profile.md` against the Progress
    Rules in `SKILL.md` (kebab-case topic keys, mutually-exclusive hint
    streaks, `review_due` staying a subset of `topic_confidence`, exercise
    slug format, profile file location). Fast enough for CI on every push.
  - `tests/scenarios/*.md` — fixed, repeatable session scripts (not
    open-ended exploration), one per bug class found and fixed in 1.2.0:
    onboarding/profile location, topic-key derivation and reuse, toolchain
    failure vs. learner bug, hint-streak scoping across a phase boundary
    plus decline-resets-streak, and last-exercise track completion.
    `tests/run_scenario_prompt.md` is the reusable agent prompt that plays
    both Sage and the scripted learner, executes real commands, and grades
    against each scenario's assertion checklist. See `tests/README.md`.
- **`tests/test_check_progress_schema.py` + `tests/fixtures/`.** The
  checker itself is plain deterministic code, so it now has unit tests
  against seven fixtures (a valid progress file, a valid track-completion
  state, and five invalid variants each isolating one violation) instead of
  relying on an expensive Tier 2 run to notice a bug in it — which is
  exactly how the `next_up`/`warn()` bug below shipped in the first place.
  Verified the new test would have caught that exact bug by reverting the
  fix and confirming it fails.
- **`.github/workflows/tier1-checks.yml`.** Runs the Tier 1 checker's unit
  tests and a plugin-manifest sanity check on every push/PR to `main` — Tier
  1 is now an actual automated gate, not a script someone has to remember to
  run.
- **Pre-release checklist in `tests/README.md`/`CONTRIBUTING.md`.** Bumping
  the plugin version now requires running all five Tier 2 scenarios (not
  just the ones nearest the change) and recording the result in the
  CHANGELOG entry, so "the harness was run" is checkable later.

### Fixed
- **Found by actually running the harness against the live spec (scenarios
  02, 03, 05 — see `tests/scenarios/`):**
  - Progress Rule 1 ("Checkpoints are explicit. Write ONLY on `/checkpoint`
    or learner confirmation") read as an absolute gate on every field write,
    contradicting Rules 5/6/8 and Step 6b, which already mandate immediate,
    unconditional writes for `completed_exercises`, `topic_confidence`/
    `review_due`, and the hint streaks. Reworded to make explicit that Rule 1
    governs the narrated full-file save, not the field-level writes other
    rules already require — an interrupted session shouldn't lose those.
  - `tests/check_progress_schema.py`'s `next_up: null` advisory was
    implemented as `warn(label, True, detail)` — since `warn()` only emits
    `WARN` on a falsy condition, this line could never print anything but
    `PASS`, silently suppressing the "verify track completion manually"
    reminder in exactly the case it exists to flag. Added a dedicated
    `note()` helper for unconditional advisories and switched this check to
    use it.

## [1.2.0] — 2026-07-01

### Added
- **Execution-based verification (Step 6b).** Exercises are no longer graded by Sage reading the code — a new `verify` field on the curriculum YAML header (a shell command template, or `manual` for non-executable exercises) is actually run via Bash before an exercise can enter `completed_exercises`. Closes the gap between the "no black boxes" principle and Sage's own grading process.
- **Axis re-calibration signal.** New `low_hint_streak`/`high_hint_streak` progress fields detect when a track's declared Mastery no longer matches reality (3 hint-free exercises in a row → offer a bump; 2 struggle-heavy exercises in a row → offer to dial back). Surfaced as an extra option in the phase-transition AskUserQuestion, never a silent change. Accepted recalibrations are written to a new `axis_overrides` field, layered on top of the curriculum's declared axes rather than mutating the curriculum file.
- **Structured learner model.** Progress file gains `topic_confidence` (per-topic solid/shaky/struggling map) and `review_due` (topics needing revisit), replacing reliance on the free-text `observations` field alone for retention tracking. `/sage-review` and `/sage-drill` now pull from `review_due` when no topic is specified, and `/sage-progress` surfaces both fields.
- **`curricula/python-basics.md`** — a complete, real 3-phase example track (Syntax & Flow → Data Structures & Functions → OOP & Persistence, building toward a small CLI task tracker), proving the template out end-to-end instead of leaving only `TEMPLATE.md`'s inline example.

### Changed
- Curriculum YAML header spec (`TEMPLATE.md`) now documents the required `verify` field.
- `/sage-review`, `/sage-drill`, `/sage-progress` command definitions updated to reflect the new progress fields.
- `/challenge` mode now runs Step 6b before checkpointing — it only skips the teaching steps (1-5), not verification.
- Step 5 (Comprehension Check) now writes to `topic_confidence`/`review_due` directly, instead of only `/review` and `/drill` touching those fields.
- Track Setup onboarding no longer triggers on "curricula/ has only TEMPLATE.md" (broken by shipping `python-basics.md`) — it now keys off whether the progress file has an `active_track`, and offers existing curricula as a starting option before falling back to the custom-track interview. That offer (Round 0) only fires during onboarding — `/sage-new-track` invoked explicitly skips it, since running that command already states custom-track intent.
- README's First Run and Adding Curriculum Tracks sections updated to match: onboarding may offer a bundled track, and manual curriculum authors need to fill in `verify`.

### Fixed
- Step 6b now specifies exactly when `low_hint_streak`/`high_hint_streak` update (immediately after verification, from that exercise's `hint_count`) — previously the fields existed but nothing said when to touch them.
- Step 6b now specifies the fallback when a curriculum has no `verify` field at all (treat as `manual`, tell the learner once) — previously only documented here in the changelog, which Sage never reads at runtime.
- Progress Rules now explicitly instruct Sage to treat missing `topic_confidence`/`review_due`/`axis_overrides`/hint-streak fields as empty/zero on old progress files — same issue, moved from changelog-only documentation into the actual spec.
- **Found via a live simulated session (an agent actually role-playing Sage and executing `verify` commands, not just re-reading the spec):**
  - `topic_confidence`/`review_due` keys were never actually defined — Progress Rules now specify a kebab-case, one-key-per-Topics-bullet convention (Rule 10).
  - `current_exercise`/`next_up` promotion timing was implied by the schema example but never stated as a rule — Step 6b now specifies it explicitly (point 8).
  - The phase-transition recalibration offer assumed streaks were phase-scoped; they're exercise-scoped and can cross a phase boundary. Reworded to "the last few exercises." Also added: declining the offer resets the triggering streak so it doesn't resurface at every subsequent transition.
  - Step 6b now sanity-checks that a failed `verify` run is actually the learner's bug and not a toolchain problem (missing interpreter, platform alias stub, PATH issue) before treating it as a Gotcha or counting it against the streaks.
  - `curricula/python-basics.md`'s `verify` command was `python3 {file}`, which resolves to a non-functional Windows Store alias stub and reported a false failure on a verified-correct solution. Changed to `python {file}`; `TEMPLATE.md`'s Tip 7 now calls this out as a concrete trap.
  - Generated learner profiles were being written to `references/learner-profile.md` — inside the plugin's own installed skill directory, which a plugin update/reinstall can wipe, and inconsistent with the project-root-scoped `.sage-progress.json`. Moved to `.sage-profile.md` in the project root; `references/learner-profile-template.md` remains the read-only structure Sage fills in from.
- **Found by re-verifying the above fixes with a second live simulation:** the topic-key rule (Rule 10) didn't cover unlabeled, comma-separated Topics bullets ("Variables, dynamic typing, truthiness") — added a rule for that shape. The exercise-pointer promotion rule (Step 6b point 8) didn't say what happens when the completed exercise was the curriculum's actual last one — added explicit track-completion handling (`next_up: null`, congratulate, offer `/tracks` or `/new-track`).

## [1.1.0] — 2026-04-25

### Changed
- **Refreshed `references/philosophy.md`** to align with the updated DEV_PHILOSOPHY master doc. The Three Axes framing and teaching-effect tables stay; the Six Principles are now in numbered form with explicit axis-conditional intensity guidance for each.
- **Tightened the "Explain before building" guidance** — added explicit plan-length expectations (three bullets often beats three paragraphs; length earned by complexity, not thoroughness theater).
- **Tightened the "Explain why" stance** — reasoning is delivered on demand or when the change is genuinely non-obvious, not by default.

### Added
- **New `## Voice and Length` section in `SKILL.md`** — names the distinction between Sage's voice *register* (warm, direct, bonfire-flavored) and *length* (terse by default, long only when earned). Output tokens cost real money in agentic loops; this section gives Sage explicit terseness discipline per command.
- **New "Operational Discipline for Sage" section in `philosophy.md`** — codifies the same length discipline at the philosophy level, plus "match the medium" guidance.

### Notes
- No breaking changes. All 20 commands, the 7-step lesson flow, the curriculum format, the learner-profile template, and the progress file format are unchanged.
- Plugin metadata (skills directory, command paths) unchanged. Existing installations can be updated in place via marketplace re-install or git pull + revalidate.

## [1.0.0] — 2026-04-06

### Added
- Initial release
- 7-step structured lesson flow (Concept → Bridge → Example → Gotchas → Comprehension Check → Exercise → Destination Connection)
- 20 `/sage-*` slash commands with descriptions
- AskUserQuestion integration throughout (onboarding, comprehension checks, exercise setup, drills, hints, phase transitions)
- Three Axes Framework integration (bundled as `references/philosophy.md`)
- Interactive onboarding: profile setup + first track creation via AskUserQuestion
- Multi-track curriculum system with `/sage-switch` and `/sage-tracks`
- Pluggable curricula with YAML header (mastery/consequence/intent calibration)
- File-based progress tracking (`.sage-progress.json`)
- Curriculum template for creating new tracks
- Learner profile template
