---
name: harbour-space-roster-fetch
description: Use when fetching the enrolled-student roster for an upcoming or current Harbour Space cohort from student.harbour.space. Triggers on "check the students list", "pull the roster", "who's in the cohort", or a `student.harbour.space/students-list/{id}` URL. Also use to fetch the matching attendance or grades view — the cohort id is shared across all three.
---

# Harbour Space — fetch cohort roster

## What it does

Pulls the enrolled-student roster (names, programs, degree level, count) for a Harbour Space cohort from the admin portal, even though the page requires authentication and `WebFetch` returns empty.

Mechanism: drive the user's Chrome through the chrome-devtools MCP. If the controlled browser doesn't have a session yet, pause once for the user to sign in, then resume.

## URL shape

Each cohort has a single integer id. The three useful admin views share that id:

| View | URL |
|------|-----|
| Students list | `https://student.harbour.space/students-list/{id}` |
| Attendance | `https://student.harbour.space/attendance-list/{id}` |
| Grades entry | `https://student.harbour.space/grades/{id}` |

If the user gives you any one of these, you have all three.

Known ids: `1349` = CS411 Barcelona May 2026 (AI Assisted DevOps).

## Procedure

1. **Try the controlled browser.** Load the `mcp__plugin_chrome-devtools-mcp_chrome-devtools__*` tools via `ToolSearch` if not already available, then `list_pages`. If only `about:blank`, proceed; an existing tab on `student.harbour.space` means the session is already there.

2. **Navigate.** `navigate_page` to `https://student.harbour.space/students-list/{id}`.

3. **Take a snapshot.** `take_snapshot` — do *not* take a screenshot, the a11y tree gives you names and programs as plain text.

4. **If the snapshot shows a login form** (heading "Sign in" / email + password fields), stop and tell the user: "Sign in inside the controlled browser, then say 'go'." Wait. When they confirm, re-snapshot.

5. **Parse the repeating block.** Each enrolled student is a 6-element run in the a11y tree:
   ```
   image "avatar" url=...person/photo/{photo-id}/...
   StaticText "{first-name-display}"        ← display first name (may include middle)
   StaticText "{first-name}"                ← duplicate, parsed surname split
   StaticText " "
   StaticText "{last-name}"
   StaticText "{program}"                   ← e.g. "Data Science", "Computer Science", "Applied Data and Computer Science", "Course based"
   StaticText "{level}"                     ← "Master", "Bachelor", "No degree"
   ```
   The roster ends when you hit a `heading level=3` (the detail-pane preview of the first student).

6. **Verify the count** matches the `"Enrolled Students (N)"` static text above the list.

7. **Emit a table** — `# · Name · Program · Level` — and a program-mix line (e.g. "10 DS Master · 5 CS Master · ...").

## Observations to surface (don't skip these)

After the table, scan for and call out:

- **Roster vs. planned cohort size.** Cross-check against memory `cs411_2026_cohort_size` (or the equivalent for other cohorts). If it's drifted, update both the memory and the cohort note in Obsidian.
- **Program mix skew.** If >40% of the cohort is *not* CS-trained (e.g. heavy Data Science), flag it — DevOps lands differently and Session-1 framing needs adjusting.
- **TA dual-listed as student.** The TA's name shows up in `42.14 Harbour Space AI Assisted DevOps 2026 BCN.md` under Class Logistics. If they also appear in the roster, raise it — affects grading treatment, pairing, oral defenses.
- **Audit-style students.** Anyone with `Course based / No degree` is not on a degree track; track engagement separately from rubric grades.
- **Recompute budgets.** Manual grading hours (≈ N × 6 challenges × 18 min), oral-defense hours (N × ~10 min), code-review pairing (N/2 pairs, or triads if N is odd at the small end).

## After fetching

- **Update memory** — `cs411_2026_cohort_size.md` (or the cohort's equivalent) with the confirmed count, program mix, and revised budgets.
- **Update Obsidian** — append a `## Confirmed Roster (verified YYYY-MM-DD)` section to the cohort's note under `42 Other Projects/42.{N} Harbour Space ... .md`. Include the source URL, table, program mix, and observations.
- **Don't commit the roster to git.** PII. The Obsidian vault is private; the repo isn't necessarily.

## Gotchas

- `WebFetch` against `student.harbour.space` returns an empty page (no cookies). Don't waste a turn trying — go straight to the chrome-devtools MCP.
- The chrome-devtools MCP launches a *clean* Chrome, not your daily-driver session. Login state is per-MCP-process and resets when the MCP restarts. If a sign-in is needed once, it's needed again next session.
- Names in the a11y tree are duplicated (first-name appears twice). Don't double-count by mistake when parsing.
- "Course based / No degree" is one student type, not two; don't split it.
- The admin portal also exposes detailed PII (DOB, email, country) below the list when a row is selected. Don't extract or save that unless the user explicitly asks — names + program + level are enough for delivery planning.

## Related

- Memory: `cs411_2026_cohort_size`, `harbour_space_admin`
- Skill: `harbour-space-course-delivery` (the budget math + rubric this feeds into)
- Obsidian: `42 Other Projects/42.14 Harbour Space AI Assisted DevOps 2026 BCN.md` (reference template for the Confirmed Roster section)
