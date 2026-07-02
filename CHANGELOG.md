# Changelog

## [1.7.2] — 2026-07-02

### Fixed
- **Axis fields could silently break if a session ran in a non-English
  language.** Nothing previously told Sage to keep `mastery`, `consequence`,
  and `intent` as their literal English tokens (`low`/`medium`/`high`,
  `growth`/`balanced`/`output`) when writing a curriculum's frontmatter —
  and Round 4's options are written verbatim from what the learner picks.
  A learner conversing in, say, Spanish could plausibly get a translated
  value written into the file instead, which the Three Axes calibration
  logic wouldn't recognize, silently breaking calibration with no visible
  error. Added an explicit instruction in `SKILL.md` to always persist
  those three fields in English regardless of conversation language, while
  letting the rest of the session converse naturally in whatever language
  the learner uses.

## [1.7.1] — 2026-07-02

### Fixed
- **`scripts/check_framework_drift.py` crashed on Windows exactly when it
  had something real to show.** Discovered while doing a fourth documentation
  pass over the upstream `three-axes-framework` repo: editing that repo's
  `SKILL.md` (a command-naming fix, unrelated to this project) caused genuine
  drift, and running the drift checker to confirm it crashed with
  `UnicodeEncodeError` trying to print the diff — Windows consoles default
  stdout to the system codepage (e.g. `cp1252`), which can't encode
  characters like the "↓" in SKILL.md's tier diagram. Forced UTF-8 output via
  `sys.stdout.reconfigure(encoding="utf-8")` at the top of `main()`. Verified
  against the actual pending drift: the script now prints the diff correctly,
  and the upstream change turned out to be cosmetic (command-name formatting,
  not a conceptual framework change) — accepted as the new baseline via
  `--update-snapshot` rather than requiring a `philosophy.md` edit.

### Added
- **`three-axes-framework` declared as a `plugin.json` dependency.** Claude
  Code has no mechanism for one plugin to read another's files at runtime
  (confirmed against the platform docs before building anything here — no
  `${CLAUDE_PLUGIN_ROOT}`-equivalent for a sibling plugin, no live cross-
  plugin file access), so this doesn't make `philosophy.md` a live view onto
  the standalone plugin. What it does do: installing sage-instructor now
  auto-installs `three-axes-framework` alongside it, so a learner gets the
  general always-active coding philosophy applied outside teaching sessions
  too, not just Sage's teaching-specific calibration. Requires sage-
  instructor to be listed in the same marketplace (`lux-solari-plugins`) as
  `three-axes-framework` — bare-string dependencies resolve within the
  declaring plugin's own marketplace; a companion change lists sage-
  instructor there.
- **`scripts/check_framework_drift.py`.** `references/philosophy.md` is a
  teaching-specific adaptation of the standalone plugin's framework, not a
  copy — lesson-step calibration, curriculum-generation axis inference, and
  other machinery that only exists here. A sync script that literally
  overwrote it with upstream content would destroy that adaptation. Instead
  this fetches the upstream `SKILL.md`, diffs it against a cached snapshot
  of the version `philosophy.md` was last reconciled against, and prints
  the diff so a maintainer can decide by hand whether the change matters
  for teaching contexts — never auto-overwrites. `--update-snapshot`
  accepts the current upstream content as the new baseline after manual
  reconciliation.
- `skills/sage-instructor/references/.three-axes-upstream-snapshot.md` —
  the initial baseline snapshot, captured from the upstream repo at commit
  `0f6c8db` (2026-03-25, marketplace-published as `three-axes-framework`
  v1.1.3).

## [1.6.0] — 2026-07-02

### Added
- **Grounding-research step in Track Setup.** Before generating a custom
  curriculum, Sage now judges whether the stated topic is a fast-moving
  library/framework with a versioned, changing API (vs. stable fundamentals
  training data already gets right) and, if so, uses WebSearch/WebFetch to
  pull current docs before writing a single exercise — recording what it
  consulted in a new optional `sources` YAML field (see `TEMPLATE.md`).
  Motivated by the original "use an LLM to teach me stuff" premise: a
  generated curriculum for something like Raylib is only as good as how
  current its underlying facts are.
- **Track Setup topic shortcut.** When the learner's trigger message already
  names a specific topic (e.g. "teach me Go, let's build something" instead
  of a bare `/sage-start`), Round 0 no longer asks blind: a topic matching no
  bundled curriculum skips Round 0's question entirely (and Round 1, since
  "what do you want to learn?" was already answered); a topic matching one
  compresses Round 0 into a direct confirm naming that track instead of the
  generic list. A genuinely ambiguous topic still asks normally.
- **`tests/scenarios/09-grounding-research-trigger.md`.** Tests the
  grounding-research trigger branch (C++/Raylib) — confirms a real
  WebSearch/WebFetch call happens, the `sources` field gets populated with
  genuine references, and at least one generated API element traces back to
  what was actually found (not a plausible-sounding invention).
- **`tests/scenarios/10-track-setup-topic-shortcut.md`.** Two-part scenario
  for the new topic shortcut: Part A (Elixir, no bundled match) proves the
  skip branch; Part B (Python, matches `python-basics`) proves the compress
  branch still offers — and honors — the custom-track alternative rather
  than railroading toward the match.

### Changed
- **`tests/scenarios/06-custom-track-creation.md`** gained a no-`sources`
  assertion (proving the grounding-research step correctly skips for stable
  topics), and its trigger message was reverted to topic-neutral ("Hey, I
  want to pick up a new skill — what have you got?", topic revealed at Round
  1 instead). Needed because the scenario's own regression target — Round 0
  presenting the full option list — only holds when no topic was pre-stated;
  a topic-naming trigger now legitimately changes Round 0's behavior under
  the new shortcut rule. Also swapped its example topic from Rust to Go: the
  original topic collided with the `rust-cli` curriculum added in 1.5.0,
  making Round 0 offer three options instead of the two the scenario assumed
  — a real staleness bug this session's own earlier release introduced.
- **`tests/scenarios/09-grounding-research-trigger.md`**'s trigger message
  was likewise reverted to topic-neutral for the same reason, keeping it
  focused on one regression target (grounding-research) rather than
  entangling it with the new shortcut rule.

### Verified
- **`09-grounding-research-trigger`**: 10/10 PASS. Real tool calls confirmed
  (WebSearch + two WebFetch calls against raylib.com and its GitHub wiki);
  the generated `verify` command's linker flags and the exercises' API
  calls (`InitWindow`, `DrawCircle`, `RAYWHITE`, ...) all traced back to the
  actual fetched content, including catching that Raylib is currently on
  v6.0 — a genuine grounding signal, not a coincidence.
- **`10-track-setup-topic-shortcut`**: 10/10 PASS (5/5 each part). Skip and
  compress branches both confirmed; picking custom despite a compressed
  match correctly avoided defaulting to the bundled track.
- **`06-custom-track-creation`** (re-run after the Go/topic-neutral fix):
  9/9 PASS.
  - Soft findings not acted on this pass: the "genuinely ambiguous" branch
    of the topic-shortcut rule has no worked example and remains untested —
    candidate for a future scenario if the shortcut sees real use; the
    grounding-research rule doesn't say how many sources are "enough" to
    prove a claim was checked rather than assumed; a scratch-test-generated
    curriculum's `verify` command was Linux-flag-specific with no
    `prerequisites` callout — a curriculum-quality nit in generated output,
    not a `SKILL.md` defect.
- Pre-release checklist (`tests/README.md`/`CONTRIBUTING.md`) updated from
  "all eight" to "all ten" scenarios.

## [1.5.0] — 2026-07-02

### Added
- **`skills/sage-instructor/curricula/rust-cli.md`.** Second bundled
  curriculum — Rust CLI fundamentals building toward Ferrogrep, a
  production search tool. Deliberately declares axes sharply different
  from `python-basics` (`mastery: medium, consequence: high, intent:
  output` vs. `low, low, growth`) so `/sage-switch` has an actual
  posture change to get right or wrong, not just a second copy of the
  same shape.
- **`tests/scenarios/08-multi-track-switching.md`.** `/sage-tracks` and
  `/sage-switch` had only ever existed alongside a single real curriculum —
  never exercised with a second one actually present (see #2's "out of
  scope for this pass" list). Checks progress isolation, hint-streak
  isolation, and axis-posture isolation between two concurrently-tracked
  courses, plus `/tracks` status accuracy across both.

### Fixed
- **Found while drafting scenario 08, before the live run:** `SKILL.md`'s
  Track Management section gave `/tracks` and `/switch` one line each —
  thin enough that status vocabulary (`active` vs. `started` vs. `not
  started`) and switch semantics (resume an existing entry vs. initialize a
  fresh one; never re-run Track Setup onboarding on switch) were only
  inferable by chaining together Progress Rule 9 and Track Setup's Round-0
  gating language, not stated directly. Added an explicit paragraph
  spelling out both.

### Verified
- **`08-multi-track-switching`**: 8/8 PASS on the first live run. Progress,
  hint-streak, and axis-override isolation between `python-basics` and
  `rust-cli` all held under a live switch-and-back; teaching posture during
  the rust-cli exercise was confirmed to match its own declared axes
  (compressed concept+bridge, explicit no-black-boxes framing on the borrow
  checker, efficient Steps 1-4), not leaked from python-basics.
  - Toolchain limitation, disclosed not hidden: the run's machine had no
    Rust toolchain installed, so the one rust-cli exercise was verified by
    manual trace instead of a real `rustc` run — same class of gap Step
    6b.3 already carves out as not counting against the exercise, but
    graders of that specific transcript should know it's compiler-unverified.
  - Soft finding, not acted on: rust-cli's `verify` command's `/tmp/`
    temp-file path assumes a Unix-like shell. Left as-is — it matches the
    existing convention in `TEMPLATE.md`'s own C++ example and the curriculum
    scenario 06 generated live, and Sage's actual execution environment
    (Git Bash) already provides a working `/tmp`.
- Pre-release checklist (`tests/README.md`/`CONTRIBUTING.md`) updated from
  "all seven" to "all eight" scenarios.

## [1.4.0] — 2026-07-02

### Added
- **`tests/scenarios/07-axis-recalibration-accept.md`.** Scenario 04 only
  ever exercised declining the recalibration offer; the **accept** branch
  (write to `axis_overrides`, confirm the change) had never been live-tested.
  This scenario triggers `low_hint_streak >= 3` (the "Mastery is probably too
  low" signal, complementing 04's `high_hint_streak` coverage), accepts the
  bump, then drives a second streak past threshold to confirm a later offer
  is measured against the newly-applied level — a follow-up named in #2's
  "out of scope for this pass" list.

### Fixed
- **Found while drafting scenario 07, before the live run:** the accept
  branch of Axis Re-Calibration said to write `axis_overrides` and confirm
  the change, but — unlike the decline branch — never said whether the
  triggering streak resets. Left unresolved, an unreset streak could
  immediately re-fire the same offer against the level Sage had just
  applied. Added a rule: accepting resets the streak too, same as declining;
  only a fresh streak against the new level earns a fresh offer.
- **Found by the scenario 07 live run:** the phase-transition question's 5th
  option (the recalibration offer) didn't say what happens to the other 4
  options when it's picked — is it a standalone choice, or does it also mean
  "start Phase N+1"? The live run inferred the latter (matching the
  scenario's own script), but SKILL.md didn't say so. Added a rule: picking
  the 5th option applies the recalibration and proceeds as if "Start Phase
  N+1" had been picked; the standard menu isn't asked again afterward.

### Verified
- **`07-axis-recalibration-accept`**: 9/9 PASS on the first live run. Real
  Python exercise scripts written and executed for real via `python`
  (not `python3` — the machine's `python3` is a Windows Store stub, exactly
  the toolchain-vs-learner-bug gotcha scenario 03 guards against); progress
  file diffed at every checkpoint; `curricula/python-basics.md` on disk
  confirmed unchanged (`git status`/`git diff` empty) throughout, i.e. the
  override never leaked into the curriculum source.
  - Soft finding (not acted on this pass): Progress Rule 10's topic-key
    derivation example ("Variables, dynamic typing, truthiness" → `variables`)
    is a 3-term bullet reduced to its first noun; Phase 1's "Lists, dicts,
    sets, tuples" bullet is a 4-term coordinate list where none of the
    exercises isolate to just one of those structures, and it's unclear
    whether Rule 10 wants `lists` (strict first-term) or a compound key.
    Same category of ambiguity as 1.3.0's `consequence`-derivation finding,
    but orthogonal to axis recalibration — left for a future scenario/issue
    rather than guessed at here.
- Pre-release checklist (`tests/README.md`/`CONTRIBUTING.md`) updated from
  "all six" to "all seven" scenarios.

## [1.3.0] — 2026-07-02

### Added
- **`tests/scenarios/06-custom-track-creation.md`.** Every prior scenario
  (01-05) and the 1.2.1 real-install smoke test picked the existing bundled
  `python-basics` track at Track Setup Round 0. This scenario instead picks
  "Build a custom track" and scripts the full Round 1-4 interview, closing
  the last major untested path from a fresh install (see #2).

### Fixed
- **Found by the scenario 06 live run:** the Track Setup interview (Rounds
  1-4) never explicitly directs Sage to derive the curriculum's
  `consequence` axis field — only `mastery` (Round 3) and `intent` (Round 4)
  have a stated mapping. The live run got `consequence` right anyway by
  inferring it from the described project's stakes, but that was judgment,
  not a followed instruction — a spec gap that happened not to bite this
  time. Added an explicit rule: infer `consequence` from Round 2's answer
  (no project/personal project → `low`; shared or stakes-bearing → `medium`
  or `high`), rather than leaving it unspecified.

### Verified
- **`06-custom-track-creation`**: 8/8 PASS on the first live run — the first
  scenario in this harness's history to pass clean without needing a fix
  first. Confirmed via a real generated curriculum
  (`curricula/rust-cli-grep.md`) with a `verify` command
  (`rustc {file} -o /tmp/sage_out && /tmp/sage_out`) that was smoke-tested
  for real, not just read as plausible-looking text; confirm-before-save was
  checked by verifying the file didn't exist on disk between the curriculum
  being displayed and the learner confirming it.
- Pre-release checklist (`tests/README.md`/`CONTRIBUTING.md`) updated from
  "all five" to "all six" scenarios.

## [1.2.1] — 2026-07-02

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
  - The Axis Re-Calibration section's two threshold bullets were worded
    asymmetrically — only the `low_hint_streak` bullet stated "at the next
    phase transition," leaving it ambiguous whether `high_hint_streak`'s
    offer could also surface at a plain exercise-complete menu. Made
    explicit that both signals are checked only at phase-transition points.

### Verified
- **Full Tier 2 pre-release run (all five scenarios) against this
  `SKILL.md`/`curricula/python-basics.md`, live via an agent playing both
  Sage and each scripted learner:**
  - `01-onboarding-and-profile-location`: 6/6 PASS.
  - `02-topic-key-consistency`: 5/5 PASS.
  - `03-toolchain-vs-learner-bug`: 6/6 PASS.
  - `04-hint-streak-scoping-and-decline`: first run surfaced 4/5 PASS, 1
    FAIL — but the FAIL was a bug in the *scenario itself* (it asserted a
    fresh recalibration offer after `P1-comprehension-refactor`, which isn't
    Phase 1's last exercise, so per SKILL.md's phase-transition-gated
    mechanism no offer was ever going to fire there). Fixed the scenario to
    assert at `P1-json-roundtrip` (Phase 1's actual last exercise) instead;
    also tightened the Axis Re-Calibration wording asymmetry noted above.
    Re-ran end-to-end (fresh scratch project, all 7 steps replayed with real
    exercises/verify runs): 6/6 PASS, including the two new assertions that
    the offer correctly stays silent at both non-transition points and
    re-fires as a genuinely fresh signal at the real Phase 1→2 transition.
  - `05-track-completion-handling`: 5/6 PASS, 1 "FAIL" that was the
    `check_progress_schema.py` bug documented above, not a spec issue.
  - **Net result: all five Tier 2 scenarios pass cleanly against the current
    `SKILL.md` and `curricula/python-basics.md`.**
- **Real-install smoke test.** Every scenario above (and the original 1.2.0
  bug hunt) validated the spec via an agent reading `SKILL.md` directly and
  role-playing Sage — never through Claude Code's actual plugin-loading and
  skill-triggering machinery. Ran `claude -p --plugin-dir` against this repo
  from a fresh scratch project with no prior profile/progress files: the
  skill triggered correctly from unscripted natural language ("teach me
  Python, let's start," no slash command), ran Profile Setup, offered the
  bundled "Python Foundations" track via Round 0, skipped straight to Phase
  0 on selection, and produced a real lesson (Steps 1-5, correctly bridging
  to the stated Java/JS background including the `[]` truthiness
  JS-vs-Python gotcha). The resulting `.sage-profile.md` and
  `.sage-progress.json` — written by the real mechanism, not staged — pass
  `check_progress_schema.py` with 0 failures. (`AskUserQuestion` isn't
  available in headless `-p` mode; Sage correctly degraded to plain
  numbered questions instead of erroring.) `claude plugin validate .` also
  passes.

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
