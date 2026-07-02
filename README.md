# Sage Instructor

A Claude Code plugin that turns Claude into **Sage** — an adaptive programming instructor that teaches through structured courses with discovery-first pedagogy, interactive exercises, and progress tracking.

Powered by the [Three Axes Framework](https://github.com/luxsolari/three-axes-framework).

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

Teaching intensity adapts via the Three Axes:
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
claude plugin validate ./sage-instructor
```

## First Run

Type `/sage-start`. If this is your first time, Sage interviews you using AskUserQuestion to:

1. **Set up your profile** — languages you know, experience level, learning style, tone preference
2. **Pick your first track** — if the plugin ships a ready-to-go curriculum (like the bundled Python Foundations track), Sage offers it as a starting point; otherwise, or if you'd rather build your own, it interviews you on what you want to learn, your destination project, current level, and priority

After that, `/sage-start` resumes where you left off.

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

Use `/sage-new-track` — Sage interviews you and generates the curriculum.

Or manually: copy `skills/sage-instructor/curricula/TEMPLATE.md`, fill in the YAML header and phases, and submit a PR. Don't skip the `verify` field — it's the shell command Sage runs to actually check an exercise before marking it complete (or `manual` if there's nothing to run). See `skills/sage-instructor/curricula/python-basics.md` for a complete worked example.

## Companion Plugins

- **[Three Axes Framework](https://github.com/luxsolari/three-axes-framework)** — the always-active philosophy plugin that Sage builds on. Declared as a `plugin.json` dependency, so installing sage-instructor installs it automatically: you get the general framework applied to *all* development work, not just learning sessions, on top of Sage's teaching-specific calibration.
- `skills/sage-instructor/references/philosophy.md` is **not** a copy of that plugin's framework doc — it's a teaching-specific adaptation (lesson-step calibration, curriculum-generation axis inference, and other machinery that only makes sense inside a structured course). There's no live sync between the two; run `python3 scripts/check_framework_drift.py` to check whether the upstream framework has changed since `philosophy.md` was last reconciled against it (see the script's docstring for details).

## License

MIT — see [LICENSE](LICENSE).

---

*"Every language mastered is a new spell in your programming grimoire."*
