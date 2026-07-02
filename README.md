# Sage Instructor

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.7.2-informational.svg)](CHANGELOG.md)

A Claude Code plugin that turns Claude into **Sage** — an adaptive programming instructor that teaches through structured courses with discovery-first pedagogy, interactive exercises, and progress tracking.

## In Action

```
You:   /sage-start

Sage:  First time here — let's get your profile set up. (A few quick
       questions: languages you know, experience level, learning style,
       tone preference.)

       Got it. Here's what's available: Python Foundations, Rust CLI
       Tools, or build something custom — which one?

You:   Python Foundations

Sage:  Phase 0: Syntax & Flow. Let's start with variables and control flow —
       coming from Java, the big shift is Python has no `var`/type
       declarations, and indentation IS the block structure, not a style
       choice.

       Here's the shape:

           def classify(n):
               if n % 15 == 0:
                   return "FizzBuzz"
               ...

       Quick check before you build it — what does `n % 15 == 0` catch
       that checking `n % 3 == 0 and n % 5 == 0` separately would also
       catch?

       Your turn: write `P0-fizzbuzz`. I'll watch, not solve.
```

That's the shape of every lesson — concept, a bridge from what you already know, a runnable example, a comprehension check before you touch the keyboard, then you build it.

## What it does

Sage delivers structured learning tracks through a consistent 7-step lesson flow:

| Step | Name | What happens |
|---|---|---|
| 1 | **Concept** | The what and why — plain language first |
| 2 | **Bridge** | Connect to languages you already know |
| 3 | **Code Example** | Small, complete, runnable, annotated |
| 4 | **Gotchas** | Where your existing habits will mislead you |
| 5 | **Comprehension Check** | AskUserQuestion quiz before the exercise |
| 6 | **Exercise** | You build it — scaffolds, not solutions |
| 7 | **Destination Connection** | Why this matters for your project |

("AskUserQuestion" above is Claude Code's native interactive multiple-choice prompt — Sage uses it throughout for profile setup, comprehension checks, hints, and drills, referenced by name elsewhere in this README.)

Teaching intensity adapts via the [Three Axes Framework](https://github.com/luxsolari/three-axes-framework) (a separate always-active philosophy plugin Sage builds on — more in [Companion Plugins](#companion-plugins)):
- **Mastery** — how well you know the topic → controls lesson depth
- **Consequence** — what breaks if you get it wrong → controls verification rigor
- **Intent** — learning vs shipping → controls pacing

## Installation

### Via the lux-solari-plugins marketplace (recommended)

```
# 1. Add the marketplace (one-time)
/plugin marketplace add luxsolari/lux-solari-plugins

# 2. Install the plugin
/plugin install sage-instructor@lux-solari-plugins
```

### Local development / testing

```
git clone https://github.com/luxsolari/sage-instructor
claude plugin validate ./sage-instructor   # verify structure
claude --plugin-dir ./sage-instructor      # load it for a real session, no install needed
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full contributor workflow, including running the regression tests.

## First Run

Type `/sage-start`. If this is your first time, Sage interviews you using AskUserQuestion to:

1. **Set up your profile** — languages you know, experience level, learning style, tone preference
2. **Pick your first track** — Sage offers the bundled tracks (see below) plus "build a custom track." If you already said what you want to learn in your opening message (e.g. "teach me Go, let's build something"), Sage skips the redundant question instead of asking you to repeat yourself — it only asks Round 0 straight when your topic doesn't match a bundled track, or names the match directly when it does.

After that, `/sage-start` resumes where you left off.

## Bundled Tracks

Each track builds toward a real (if fictional) project — its "destination" — so every exercise has a concrete reason to exist, not just an abstract drill.

| Track | Destination project |
|---|---|
| **Python Foundations** | Taskwright — a CLI task tracker with persistent storage |
| **Rust CLI Tools** | Ferrogrep — a production Rust CLI search tool |

Don't see what you want to learn? `/sage-new-track` builds a custom one — see [Adding Curriculum Tracks](#adding-curriculum-tracks).

## Requirements

- A working [Claude Code](https://claude.com/claude-code) install.
- Per-track toolchain: Python Foundations needs `python`/`python3` on PATH; Rust CLI Tools needs `rustc`. Custom tracks state their own prerequisites — Sage checks before generating exercises. `/sage-tracks` and each curriculum's `prerequisites` field show what's needed before you commit to one.
- Sage writes two files to your project root as you go: `.sage-progress.json` (phase, completed exercises, hint streaks) and `.sage-profile.md` (your learner profile). Nothing else on disk is touched.

## Commands

All commands are prefixed with `sage-` and appear in Claude Code's `/` autocomplete.

### Navigation
| Command | Description |
|---|---|
| `/sage-start` | Begin or resume (runs setup on first use) |
| `/sage-next` | Advance to the next lesson or exercise |
| `/sage-phase N` | Jump to a specific phase |
| `/sage-exercise NAME` | Start a specific exercise |

### Progress
| Command | Description |
|---|---|
| `/sage-progress` | Full progress report |
| `/sage-checkpoint` | Save current progress to disk |
| `/sage-recap` | Summarize current phase concepts |
| `/sage-status` | One-line status |

### Learning Modes
| Command | Description |
|---|---|
| `/sage-lesson [topic]` | Structured 7-step lesson |
| `/sage-challenge` | Problem-first — exercise with no upfront teaching |
| `/sage-review topic` | Revisit a covered topic |
| `/sage-drill` | Quick-fire AskUserQuestion comprehension rounds |

### Interaction
| Command | Description |
|---|---|
| `/sage-hint` | Escalating hints (choose depth via AskUserQuestion) |
| `/sage-explain X` | Deep dive on a concept |
| `/sage-stuck` | More direct guidance |

### Track Management
| Command | Description |
|---|---|
| `/sage-tracks` | List available learning tracks |
| `/sage-switch TRACK` | Switch active track |
| `/sage-new-track` | Create new curriculum interactively |

### Meta
| Command | Description |
|---|---|
| `/sage-help` | Show all Sage commands |
| `/sage-reset` | Clear progress |

## Adding Curriculum Tracks

Use `/sage-new-track` — Sage interviews you and generates the curriculum. For fast-moving libraries or frameworks (a game/UI library, a cloud SDK, a build tool), Sage checks current official docs before writing exercises instead of relying on training data that might be stale or reference a renamed API — stable fundamentals (a language's own core syntax) skip that check.

Or manually: copy `skills/sage-instructor/curricula/TEMPLATE.md`, fill in the YAML header and phases, and submit a PR. Don't skip the `verify` field — it's the shell command Sage runs to actually check an exercise before marking it complete (or `manual` if there's nothing to run). See `skills/sage-instructor/curricula/python-basics.md` or `skills/sage-instructor/curricula/rust-cli.md` for complete worked examples.

## Companion Plugins

- **[Three Axes Framework](https://github.com/luxsolari/three-axes-framework)** — the always-active philosophy plugin that Sage builds on. Declared as a `plugin.json` dependency, so installing sage-instructor installs it automatically — you get the general framework applied to *all* development work, not just learning sessions, on top of Sage's teaching-specific calibration. No extra setup needed.

## Contributing

Bug reports, curriculum tracks, and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for how the pieces fit together (including the regression-test harness and how `philosophy.md` relates to the Three Axes Framework above).

## License

MIT — see [LICENSE](LICENSE).

---

*"Every language mastered is a new spell in your programming grimoire."*
