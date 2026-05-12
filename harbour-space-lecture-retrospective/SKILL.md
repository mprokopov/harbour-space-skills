---
name: harbour-space-lecture-retrospective
description: Use when analyzing a past Harbour Space cohort's Google Meet lecture recordings to inform the next delivery — pacing gaps, student confusion clusters, time-per-unit vs the planned iximiuz curriculum. Scoped to CS406/CS411-style 3-week / ~15-session deliveries with unit-*.md files as the canonical topic taxonomy.
---

# Harbour Space — lecture retrospective analysis

## Overview

After each Harbour Space cohort, the Google Meet recordings accumulate in Drive. Turning ~40 hours of audio into actionable pacing / confusion / coverage insights for the next cohort is a repeatable pipeline, not a one-off script. This skill captures the playbook used to analyze the CS411 2025 (May–Jun) delivery while prepping CS411 Barcelona 2026.

**Three-phase pipeline:**

1. **Transcribe** — Drive video → ffmpeg audio → faster-whisper text.
2. **Topic-map** — parallel subagents label 5-min windows against the `unit-*.md` taxonomy.
3. **Confusion-map** — parallel subagents extract student questions + instructor re-explanations.

All three output JSON files that a small `aggregate_*.py` folds into a Markdown report.

**Reference implementation:** `lecture-analysis/` subdirectory in the `42.11 Harbour Space 2025 BKK` repo. `transcribe.py`, `analyze_topics.py`, `aggregate.py`, `aggregate_questions.py` — copy and adapt.

## When to use

- End of a cohort, planning the next one — what took too long, what got rushed, which concepts kept not landing
- Before revising a syllabus — evidence-based, not memory-based
- Writing pre-class setup guides — surface the tooling-debug questions students actually hit
- Designing a pre-recorded FAQ — rank concepts by cross-session recurrence

**Skip if** you only have one session to analyze (not worth the pipeline) or you don't have recording access.

## Phase 1 — Transcribe

### Setup (one-time)

```bash
brew install rclone ffmpeg
rclone config create gdrive drive scope=drive.readonly   # OAuth via browser
# Use `drive.readonly` — rclone can NEVER modify or delete recordings
```

Then `uv init` with `faster-whisper>=1.0.3` as the only dep. No OpenAI whisper — `faster-whisper` is same weights via CTranslate2, 3–5× faster, lower RAM.

### The streaming pattern (mandatory on low-disk Macs)

Peak disk with naive "download everything" = full cohort × ~500 MB each. On a 460 GB Mac that's often full. Process one video at a time:

```
download video → ffmpeg extract audio → delete video → transcribe → delete audio → next
```

Peak working set ≈ 700 MB regardless of cohort size. `transcribe.py` in `lecture-analysis/` is the reference.

### The three gotchas that waste hours if skipped

| Gotcha | Fix |
|---|---|
| **`condition_on_previous_text=True` (Whisper default) degenerates into `.` repetition on long audio after any quiet stretch** — in CS411 2025 this nuked 111 of 167 min of session 11 | Always pass `condition_on_previous_text=False` for lecture-length audio |
| Whisper wants 16 kHz mono PCM. Feeding it raw mp4 wastes cycles on internal transcode | `ffmpeg -i X.mp4 -vn -ac 1 -ar 16000 -c:a pcm_s16le X.wav` before transcription |
| Smoke-test realtime factor (15×+ on short clips) is a **lie** for continuous speech | Budget 2–3× realtime on M2 Pro with `medium` int8 + VAD. A 3h lecture ≈ 60–90 min wall time |

Recommended call:

```python
segments, info = model.transcribe(
    audio_path,
    language="en",
    beam_size=5,
    vad_filter=True,                 # skip silence → faster + cleaner
    condition_on_previous_text=False, # prevent decoder lock-in
)
```

Model size: `medium` int8 is the sweet spot on M2 Pro. `large-v3` is ~2× slower for marginal quality gain on lecture audio.

### Output per session

Write three files in `data/transcripts/session-NN_YYYY-MM-DD.{txt,srt,json}`. The `.json` (segments with start/end) is what downstream phases consume.

## Phase 2 — Topic-map against the curriculum

The planned taxonomy is already authored in `<training-folder>/unit-*.md`. Read all unit files, extract `name:` + title + bullet-list subtopics into a fixed prompt constant. Add a synthetic `off-curriculum` bucket for breaks/logistics/tangents.

**Dispatch one subagent per session.** Each gets:
- The full taxonomy inline
- The path to its transcript JSON
- The output path `data/analysis/session-NN_YYYY-MM-DD.json`
- Instructions to bucket by `int(segment["start"] // 300)` (5-min windows) and assign one dominant unit per window with a confidence rating and evidence quote

Agents return one line (`DONE: N windows, dominant unit: X`) and write the full JSON to disk. Aggregator reads the files, not the agent responses.

See `analyze_topics.py` in the reference implementation for the prompt template. 14 sessions dispatch in parallel → finishes in ~4 min wall time.

## Phase 3 — Question + re-explanation extraction

Same parallel-subagent pattern, different prompt. Each agent pulls:

- **Questions** with `at_min`, `topic_tag` (kebab-case slug), `asker`, `paraphrase`, `verbatim_snippet`, `kind` (`clarification|how-to|conceptual|tooling-debug|homework|logistics`)
- **Re-explanations** where the instructor re-taught a concept, keyed off cue phrases ("as I mentioned", "remember when", "let me re-explain", revisiting a diagram)

Output: `data/questions/session-NN_YYYY-MM-DD.json`. The aggregator scores topics by `questions × sessions-touched + 2×(re-explanations × sessions-touched)` — rewarding concepts that recurred, which is the signal for "didn't land".

**Known fragmentation:** agents independently pick topic slugs (`ssh-fingerprint` vs `ssh-fingerprinting`). Good enough for top-20 ranking; if you need precise cross-session dedup, add a canonicalization pass (one agent takes all slugs + counts, merges synonyms).

## Aggregation

Two small Python scripts — no LLM calls:

- `aggregate.py` → `_report.md` with per-unit minutes, per-session dominant topic, off-curriculum stretches
- `aggregate_questions.py` → `_confusion_map.md` with top confusion topics, question-type breakdown, sample questions per cluster

Both take under 1s on 14 sessions' worth of JSON.

## Save the output

The final `_report.md` + `_confusion_map.md` belong in Obsidian under `43 Harbour Space/`. Use the `save-to-obsidian` skill — these are course-prep inputs for next year's planning notes, not throwaway analysis.

## Quick-reference gotchas

| When you see... | Do this |
|---|---|
| Transcript ending in pages of `.` or repeated phrase | Rerun with `condition_on_previous_text=False`. Delete the broken `.txt/.srt/.json` first so the resumable check picks it up. |
| Smoke test reports 15× realtime | Don't extrapolate. Budget 2–3× for real lectures; run overnight. |
| rclone auth prompts in terminal | Use `rclone config create gdrive drive scope=drive.readonly` — one command, OAuth in browser, read-only scope is non-negotiable on lecture recordings. |
| Disk <10 GB free | Streaming pattern is mandatory. `transcribe.py` already does it. |
| Topic taxonomy is out of date | Re-read `<training-folder>/unit-*.md` — the unit files ARE the source of truth, including any new units added for the cohort being planned. |
| Considering OpenAI `whisper` | Don't. Use `faster-whisper`. Same quality, 3–5× faster, less RAM. |
| Considering using the Anthropic API for analysis | Not needed. Parallel Claude Code subagents over transcripts use the Max subscription and produce the same quality. Reserve the API for repeated batch work where Max quota would throttle. |

## Common mistakes

- **Dispatching agents before smoke-testing the prompt.** Do one agent first on the smallest session, verify the JSON, then fan out.
- **Asking agents to return the full JSON in their reply.** They should write to a file and reply with a one-line confirmation. Otherwise you burn coordinator context on 14× 5KB JSON blobs.
- **Skipping Phase 3 because Phase 2 "already labels topics".** Phase 2 tells you what was covered; Phase 3 tells you what *didn't land*. Different data, different decisions.
- **Letting topic slugs stay fragmented for a polished final report.** Fine for an internal first pass; run a canonicalization agent if you're publishing the findings.

## Related skills

- `harbour-space-course-delivery` — the delivery playbook this analysis feeds back into
- `labctl-content` / `iximiuz-fork-training` — for authoring the revised tutorials the analysis surfaces
- `save-to-obsidian` — final destination of the reports
