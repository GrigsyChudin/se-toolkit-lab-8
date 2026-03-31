---
name: observability
description: Use VictoriaLogs and VictoriaTraces to investigate errors and traces
always: true
---

# Observability Skill

Use observability MCP tools to investigate errors, logs, and traces in the system.

## Available tools

- `logs_search` — search logs in VictoriaLogs using LogsQL query
- `logs_error_count` — count errors per service over a time window
- `traces_list` — list recent traces for a service
- `traces_get` — fetch a specific trace by ID
- `cron` — schedule recurring jobs in the current chat session

## Strategy

### For "Any errors in the last X minutes?"

- Start with `logs_error_count` for a quick overview
- Use `logs_search` to find specific error details and extract `trace_id` from error logs
- Use `traces_get` with the `trace_id` to see the full request flow and identify where it failed
- Summarize findings concisely — don't dump raw JSON

### For "What went wrong?" (failure investigation)

When the user asks about a failure, follow this investigation flow:

1. **Check recent errors**: Call `logs_error_count(minutes=10)` to see if there are recent errors
2. **Search error logs**: Call `logs_search(query='_time:10m severity:ERROR', limit=10)` to get details
3. **Extract trace_id**: From the error logs, find a `trace_id` field
4. **Fetch the trace**: Call `traces_get(trace_id="...")` to see the full span hierarchy
5. **Identify the root cause**: Look for the failing span and its error message
6. **Summarize**: Provide a concise explanation that mentions:
   - The affected service
   - The error from logs (with timestamp)
   - The failing operation from the trace
   - The root cause if identifiable

### For "Check system health"

- Call `logs_error_count(minutes=5, service="Learning Management Service")`
- If errors found, investigate with `logs_search` and `traces_get`
- Report: "System is healthy" or "Found X errors: [details]"

## LogsQL syntax

- Time range: `_time:10m` for last 10 minutes, `_time:1h` for last hour
- Severity: `severity:ERROR` for errors, `severity:WARNING` for warnings
- Service: `service.name:"Learning Management Service"`
- Combined: `_time:10m service.name:"Learning Management Service" severity:ERROR`

## Response format

- Start with a summary: "Found X errors in the last Y minutes" or "No errors found"
- List affected services with counts
- If a trace is relevant, describe the failure point in plain language
- End with actionable insight if possible
- Never dump raw JSON — always summarize in natural language

## Example flow for "What went wrong?"

1. User: "What went wrong?"
2. You: Call `logs_error_count(minutes=10)` → "Found 5 errors in Learning Management Service"
3. Call `logs_search(query='_time:10m severity:ERROR', limit=5)`
4. Extract `trace_id` from an error log
5. Call `traces_get(trace_id="e1b567215461c57aab43d24d2ac07b20")`
6. Summarize: "The request failed at the database query step — PostgreSQL connection was closed. The backend returned a 404 error, but the real issue is the database connection failure."
