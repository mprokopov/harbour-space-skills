# Harbour Space — teaching skills

A small set of [Claude Code](https://docs.claude.com/en/docs/claude-code) / Claude Agent SDK **skills** that capture the operational side of delivering a three-week intensive DevOps course at [Harbour.Space University](https://harbour.space). They're written for my own workflow (CS406, CS411) but should adapt cleanly to any short, project-based course that uses an LMS for the roster and a coding-lab platform for assignments.

If you teach at Harbour Space — or anywhere with a similar shape — feel free to fork, adapt, and copy what's useful. PRs and issues welcome.

## What's a skill?

A **skill** is a markdown file (`SKILL.md`) with YAML frontmatter that tells Claude *when* to use it. When the description matches what you're doing, Claude loads the body of the skill into context and follows it. Think of them as opinionated playbooks the model can self-trigger on — not slash commands you have to remember.

Read more: [Anthropic's skill docs](https://docs.claude.com/en/docs/agent-sdk/skills).

## What's in here

| Skill | What it does |
|---|---|
| [`harbour-space-course-delivery`](harbour-space-course-delivery/SKILL.md) | The big-picture playbook: three-stage grade flow (lab platform → Classroom → Harbour Space admin), layered assessment rubric (Core / Stretch / Debug / Reflection / Code review), agent-resistance mechanisms (oral defenses, paper exam Part 1, debug tracks), narrative grade-comment style. |
| [`harbour-space-lecture-retrospective`](harbour-space-lecture-retrospective/SKILL.md) | After-cohort analysis pipeline: Google Meet recordings → ffmpeg → faster-whisper transcripts → topic-mapped against your `unit-*.md` taxonomy → confusion-cluster report. Three-phase JSON-aggregating pipeline; reference implementation linked. |
| [`harbour-space-roster-fetch`](harbour-space-roster-fetch/SKILL.md) | Pull the enrolled-student roster from `student.harbour.space` via the chrome-devtools MCP (handles the empty-`WebFetch`-on-auth-pages gotcha). Auto-flags: DS-vs-CS program skew, TA-listed-as-student, audit-track students, recomputed grading/oral-defense hour budgets. |

## How to use

These are designed to live in `.claude/skills/<name>/SKILL.md` inside your course-materials repo (project-scoped) or in `~/.claude/skills/<name>/SKILL.md` for global use. Claude Code auto-discovers them.

```sh
# project-scoped (recommended — keeps the skill set with the cohort it serves)
mkdir -p .claude/skills
git clone https://github.com/mprokopov/harbour-space-skills.git /tmp/harbour-space-skills
cp -R /tmp/harbour-space-skills/harbour-space-* .claude/skills/
```

Or just copy the individual `SKILL.md` files you want.

## Adapting to your course

These reference my specific stack — [iximiuz Labs](https://labs.iximiuz.com) for tutorials/challenges, Google Classroom for homework collection, `student.harbour.space` as the admin portal — but the structural moves transfer:

- **Three-stage grade flow** generalizes to: where students submit → where you grade → where the institution wants the final number. Mine are iximiuz / Classroom / Harbour Space admin. Yours might be GitHub Classroom / Moodle / Banner.
- **Layered rubric** (Core completion + Stretch + Debug + Reflection + Code review) is platform-agnostic; the percentages encode my agent-resistance philosophy.
- **Roster-fetch pattern** (chrome-devtools MCP to drive an authenticated tab) works on any admin portal that returns nothing useful to `WebFetch`.

The `harbour-space-course-delivery` skill is the most opinionated — it bakes in design decisions about how to teach DevOps in the age of coding agents (debug tracks, oral defenses, paper exam, two-tier reflections). The other two are mostly mechanics.

## Background

Some context that informed the design:

- **Why agent-resistance matters.** When students can paste any rubric into an LLM and get a 90% answer, completion-only grading stops measuring understanding. The rubric and grade-flow in `harbour-space-course-delivery` are built to keep humans in the loop where it counts.
- **Why a lecture retrospective.** Memory is unreliable; pacing intuitions drift. Running Whisper over the recordings and topic-mapping them gives evidence-based input into the next syllabus.
- **Why a separate roster-fetch skill.** Cohort size confirmation drives multiple downstream budgets (grading hours, oral-defense block size, pair vs. triad code-review). It happens once per cohort but is high-leverage when it does — worth codifying.

## License

MIT. Use these however you like. Attribution appreciated but not required.

## Contact

Maksym Prokopov — [@mprokopov](https://github.com/mprokopov) on GitHub, [faculty profile](https://harbour.space/faculty/maksym-prokopov) at Harbour Space.

If you adapt these for your own course, I'd love to hear what you changed and why.
