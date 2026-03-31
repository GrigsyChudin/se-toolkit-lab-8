---
name: lms
description: Use LMS MCP tools for live course data
always: true
---

# LMS Skill

Use LMS MCP tools to answer questions about course data, student performance, and lab analytics.

## Available tools

- `lms_health` — check backend health and item count; call first if unsure the backend is up
- `lms_labs` — list all available labs; call before any lab-specific query when the lab is unknown
- `lms_learners` — list all registered learners
- `lms_pass_rates(lab)` — pass rates and average scores per task for a specific lab
- `lms_timeline(lab)` — submission timeline (date + count) for a specific lab
- `lms_groups(lab)` — average score and student count per group for a specific lab
- `lms_top_learners(lab, limit)` — top learners by average score for a specific lab
- `lms_completion_rate(lab)` — completion rate (passed / total) for a specific lab
- `lms_sync_pipeline` — trigger data sync; call if data seems missing or stale

## Strategy

- When the user asks about scores, pass rates, completion, groups, timeline, or top learners without naming a lab, call `lms_labs` first.
- If multiple labs are available, present them as a choice using the shared structured-ui skill: use each lab title as the user-facing label and the lab identifier as the value.
- Use type "choice" to present the lab selection on supported channels; fall back to a plain text list if the interactive tool is unavailable.
- Once the lab is known, call the relevant tool directly without asking again.
- If the backend returns an error, suggest calling `lms_sync_pipeline` and retrying.

## Formatting

- Show percentages with one decimal place (e.g. 73.4%).
- Show counts as plain integers.
- Keep responses concise — use bullet lists or short tables rather than long prose.
- When the user asks what you can do, explain: I can query live LMS data — labs, learners, pass rates, timelines, group performance, top learners, and completion rates.
