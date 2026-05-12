---
name: harbour-space-course-delivery
description: Use when preparing, delivering, or wrapping up a Harbour Space DevOps course cohort (CS406, CS411, etc.). Covers the three-stage grade flow (iximiuz → Google Classroom → Harbour Space admin), the layered assessment rubric, agent-resistance mechanisms (orals, debug tracks, paper exam Part 1), authoring-workload patterns, and the final narrative-comment style.
---

# Harbour Space DevOps course — delivery playbook

## Overview

Maksym's DevOps course at Harbour Space (CS406 Bangkok, CS411 Barcelona, etc.) runs annually. Each delivery is 3 weeks / 45 hours / 6 ECTS, cohort ~10–20 students, with interactive labs on iximiuz. This skill captures the end-to-end delivery pattern: design assessment, author content, set up Classroom, grade during the course, and land final grades in the Harbour Space admin portal.

**Pedagogical philosophy** (three principles — full version at `~/Obsidian/Remote Vault/26 Courses & Teaching/21.45 Teaching Approach - Applied DevOps Courses.md`):

1. **Fundamentals-first, not tools-first** — teach DevOps as a systems discipline (boot, memory, binaries → containers → orchestration → cloud), not a tooling curriculum. A graduate should understand *why* a container image is the shape it is, not just how to write a Dockerfile.
2. **Artifact-centric framing** — the whole course arc is "deliver the built artifact to the target environment." CI/CD is a *consequence* of the artifact model, not the subject. Tools change; the artifact → environment pipeline doesn't.
3. **Transfer, not recall, is the real assessment** — homework in one language (Go: compiled, predictable), exam in another (Node.js: dynamic, different build model). A student who memorized Go steps fails the exam; a student who internalized the principles transfers.

Subsidiary: **AI as a first-class tool.** Expected and encouraged on assignments. Agent-neutral — students use whatever they have. The language-change transfer test (principle 3) is what keeps AI use honest; the agent can solve the homework but can't shortcut understanding of what an artifact *is* when the language changes at exam time.

## When to use

- Starting to plan a new delivery (new cohort, new syllabus variations)
- Mid-course and grading the first home assignment — need the rubric template
- Final day of a cohort — writing per-student narrative comments for the admin portal
- Reviewing assessment design (orals, debug tracks, paper exam Part 1) before proposing changes
- Authoring a new tutorial and need to remember it's instructor-demoed, not student self-study

**Skip if** the task is purely content authoring mechanics (use `iximiuz-fork-training` / `labctl-content` instead).

## Daily class rhythm

Each 3-hour in-person session runs the same shape. Tutorials are **instructor demo scripts**, not student self-study:

```
15–20 min   Recap of yesterday
90–120 min  Instructor demos the day's tutorial live, narrating the WHY,
            using a coding agent in front of the class, pausing at AVD
            callouts to invite class discussion
1–2 breaks  Interleaved
10 min      End-of-day recap — what was covered, what shows up on the challenge
```

At home: students solve the day's challenge (if any) — their individual reinforcement. This is where understanding gets stress-tested.

**Implications for content design:**
- Tutorials must solve cold in ≤ 90 min so the instructor has headroom for narration, Q&A, and agent beats in a 2h demo slot.
- AVD callouts are short, optional pause-beats for the live demo, not long solo-read prompts.
- **Checkpoint markers** at 2–4 natural milestones per tutorial give code-along students who fell behind a resync point and watchers-ahead a place to wait. Shape: *"✓ Checkpoint — at this point you should have <concrete verifiable thing>."*
- **Audience framing** at the top of each tutorial makes code-along-vs-watch an explicit choice, not a silent struggle: *"You're welcome to code along, or just watch — both work. The split-attention effect is real: pick one. The URL is here for you to replay at home."*
- Instructor uses an agent in every demo — students see evaluation-in-action as implicit training, on top of the explicit Session 1.5 lesson.
- Tutorials include a `<!-- TEACHING NOTES -->` HTML comment at the top for instructor-only cues (raw solve time, key beats, agent-demo moment, anticipated stumble points, **pace advisory** — where to slow down for hands-on, where to speed through narrative).

## The three-stage grade pipeline

```
iximiuz Labs              Google Classroom              Harbour Space Admin
───────────────           ──────────────────            ───────────────────
auto-verification    →    per-assignment rubric   →    final numeric grade
(tasks: frontmatter)      grading + submissions         + narrative comment
                                                         (final deliverable)
```

- **iximiuz** — correctness oracle. Platform's `tasks:` / `hintcheck:` / `failcheck:` verifies student work automatically. `labctl api` can pull progress programmatically (endpoint per delivery; browser DevTools on author progress view reveals the path).
- **Google Classroom** — rubric-based grading during the course. Students submit an iximiuz URL + reflection text + artifact; instructor applies rubric; Classroom computes weighted totals.
- **Harbour Space admin** — `https://student.harbour.space/grades/<cohort-id>` (e.g. `/grades/1206` for CS406BKK). Enter final grade (0–100) + 2–4 sentence narrative per student. This is the course's final deliverable.

## Automation layer (scripts + config-as-code)

Classroom setup and grading run through Python scripts driven by YAML in the repo. Source of truth is `classroom/*.yaml`; scripts live in `scripts/classroom/`; setup walkthrough at `docs/automation/classroom-setup.md`.

Four scripts:

- `auth.py` — one-time OAuth2 login, token refresh thereafter
- `bootstrap.py` — YAML → Classroom API; creates course shell, 10 assignments, 15 material posts, rubric scaffolding. Idempotent.
- `grade.py` — pulls iximiuz completion per student per challenge via `labctl api`, computes auto-gradable dimensions (Core + Stretch + Debug = 65 of 100), patches Classroom draft grades. Reflection + Code review remain manual.
- `roster.py` — CSV export of enrolled students.

What this saves during delivery: ~15h over 3 weeks (Classroom setup ceremony collapsed to 15 min; per-challenge grading cut from 4.5h to ~1h because core/stretch/debug are auto).

The iximiuz endpoint path gets filled in after W1 Task 2 (progress API reconnaissance via browser DevTools on the author progress view).

## Assessment structure

### Course-wide weights

Designed to give understanding-assessment mechanisms (orals + paper + debug + code review) direct weight rather than burying orals inside a "participation" bucket. Attendance dropped — orals catch genuine non-engagement more reliably than headcount.

| Component | Weight | Mechanism |
|---|---|---|
| Weekly reflections × 3 | 10% | Classroom text submission, Friday deadlines, pass/revise |
| **Orals × 3** | **15%** | In-lab 5-min per student per week, pass/refine/fail |
| Home assignments (6 challenges) | 35% | iximiuz + Classroom rubric per challenge |
| Final exam Part 1 (paper) | 15% | 30 min closed-device, concept recall + code reading |
| Final exam Part 2 (capstone) | 25% | 2–3h, agent allowed, rubric-graded |

Directly-understanding-assessed work (orals + paper + debug + code review within challenges) totals ~42% of course grade.

### Per-challenge rubric (home assignments, 100 pts) — same for all 6

| Criterion | Weight | Evidence |
|---|---|---|
| Core verification passed | 30 | iximiuz URL (Classroom link field) |
| Stretch tasks (2–3 per challenge) | 10 | Per-task status on iximiuz challenge page |
| Debug track (all 6 challenges) | 25 | iximiuz verification on a parallel debug path |
| Reflection backed by PROMPTS.md | 15 | Classroom text submission + a mandatory `PROMPTS.md` file documenting the student's coding-agent session (or thinking process if no agent used). Thin PROMPTS.md = needs-revision. |
| Artifact code review | 20 | Student pastes artifact; instructor reviews every submission |

**Every home-assignment challenge has a debug track.** An earlier design had debug on only 3 of 6; that left half the assignments agent-speed-runnable (Core 45 + Stretch 15 = 60 auto-gradable points). Always design debug tracks in.

### Weekly reflection (×3, 5% each — part of 20% participation)

Classroom text assignment, due Friday. ~200 words:
- Two concepts learned this week I can explain
- One concept still fuzzy
- One question I asked a coding agent and the key thing its answer revealed

Graded **satisfactory / needs revision** (re-submittable once).

### Final exam two-part

- **Part 1: Paper, closed-device, 30 min, 25% of exam grade** — 10 short-answer concept recall + 2 code-reading exercises.
- **Part 2: Capstone, agent allowed, 2–3h, 75% of exam grade** — full pipeline ship (Jenkinsfile + Dockerfile + K8s manifests + observability).

## Agent-resistance — what makes this assessment work

AI is expected and encouraged on primary/supplemental assignments. What keeps grading honest about understanding:

1. **Debug tracks** on 3 challenges (Docker, K8s, CI/CD basics). Init-task seeds a broken artifact with 3 subtle bugs. Verification checks specific fixes. Agent can suggest; student must identify which suggestion is right.

2. **Weekly oral checkpoints** (5 min × 15 students × 3 weeks ≈ 4h). Three questions per student: "show me one thing you built this week"; "what does this line do"; "what if I changed X to Y?" Graded pass/refine/fail — contributes to 20% participation.

3. **Paper exam Part 1** (30 min, closed device). Concept recall + reading 3 subtle issues out of a provided artifact. Cannot be faked with an agent.

Under this design, a student who agent-speed-runs everything but doesn't understand the output scores 40–60, not 100. A student with genuine understanding plus smart agent use scores 85–100.

## Authoring workload (pre-course prep)

~80 hours across 6 weeks before class start. Breakdown:

| Task | Hours |
|---|---|
| Phase A — playground smoke + Dockerfile version bump | 6 |
| Phase B — tutorial dry-run punch list | 4 |
| AVD callouts ("Ask, Verify, Deepen") on existing tutorials | 15 |
| New tutorials (Docker deep, K8s deep, Helm intro) | 11 |
| New challenge (ArgoCD one-shot) | 3 |
| Phase C — challenge verification audit + debug tracks on 3 challenges | 12 |
| Phase D — agent rehearsal | 1 |
| Classroom setup (assignments + rubrics) | 5 |
| Final exam Part 1 paper question bank | 3 |
| Final exam Part 2 capstone rework | 4 |
| Weekly reflection prompt templates | 1 |
| Session 1.5 "Working with coding agents" lecture | 3 |
| Lecture notes / slides (new or restructured sessions) | 12 |
| **Total** | **~80** |

Slack: if W5 is tight, cut slide deliverable from 12h to 6h (half the sessions lean on tutorials as notes).

## Grading workload (during class, 3 weeks)

~40 hours total at a 15-student cohort.

| Activity | Hours/week | Total |
|---|---|---|
| Weekly reflection grading | 0.5 | 1.5 |
| Weekly oral checkpoints (5 min × 15 students) | 1.3 | 4 |
| Challenge rubric grading (2/wk × 15 × 3 min) | 1.5 | 4.5 |
| Full code reviews (2/wk × 15 × 15 min) | 7.5 | 22 |
| Final exam (paper + capstone) | — | 7 (W3) |
| Harbour Space admin entry (final day) | — | 1.5 |
| **Total** | **~10.8** | **~40** |

## Final-day Harbour Space admin entry

On the last day:

1. Open Classroom gradebook; note weighted total per student.
2. Open `https://student.harbour.space/grades/<cohort-id>` (cohort ID assigned by Harbour Space).
3. For each student, write a **2–4 sentence narrative comment** in this voice:
   - Names the student directly ("Levan proves to possess…", "Well done, Nyen!")
   - Notes 1–2 specific observations (engagement, performance pattern, context)
   - Ends with encouragement ("Keep it up!", addressed by name)

Example from CS406BKK 2025:
> *"Levan proves to possess good knowledge and skills. Exam for him wasn't such a challenge as for other participants. Keep it up, Levan!"*

Budget ~5 min per student. ~1.5h for 15 students.

## Student feedback themes (2025 BKK → 2026 BCN)

From CS406BKK end-of-course survey (NPS 80, 4.9/5 class and teacher):

- ✅ "iximiuz labs were a hit" — keep the interactive-lab approach.
- ⚠ "Want lecture notes/slides" — address via slide decks for new sessions.
- ⚠ "Too broad, not deep enough" (K8s, architecture) — CS411 adds sessions 9 (K8s deep) and 10 (K8s ecosystem).
- ⚠ "Want automated/linked grading" — address via rubric + Classroom + iximiuz API.
- ⚠ "Consider a sequel course" — log for future; out of current scope.

See Obsidian: `~/Obsidian/Remote Vault/42 Other Projects/Harbour Space Barcelona 2026 - Course Prep.md`.

## Reference URLs

- Course page: https://harbour.space/computer-science/courses/devops-maksym-prokopov-1064
- Faculty profile: https://harbour.space/faculty/maksym-prokopov
- iximiuz Labs: https://labs.iximiuz.com
- CS406BKK training: https://labs.iximiuz.com/trainings/harbour-space-devops-2025-bkk-b7f88197
- CS411 training: https://labs.iximiuz.com/trainings/ai-assisted-devops-2026-cs411-2dd22015
- Admin portal pattern: `https://student.harbour.space/grades/<cohort-id>`
- Student list pattern: `https://student.harbour.space/students-list/<cohort-id>`

## Related

- `.claude/skills/labctl-content/SKILL.md` — iximiuz content authoring reference
- `.claude/skills/iximiuz-fork-training/SKILL.md` — per-cohort content forking workflow
- `docs/curriculum/learning-objectives.md` — 8 course-level + 45 session-level objectives, Bloom's tagged, cross-referenced to assessment mechanisms
- `docs/superpowers/specs/2026-04-18-cs411-refresh-design.md` — current delivery spec (CS411 Barcelona 2026)
- `docs/superpowers/plans/2026-04-18-cs411-refresh.md` — task-by-task implementation plan
- `docs/automation/classroom-setup.md` — one-time OAuth setup for the Classroom automation
- Obsidian notes at `~/Obsidian/Remote Vault/42 Other Projects/` — delivery logs per cohort
