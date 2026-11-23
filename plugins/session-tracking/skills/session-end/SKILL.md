---
name: session-end
description: Close the current work session by writing a resume file, update the project summary, and close the time-tracking row(s). Use when the user asks to end, close or wrap up the session (e.g. "end the session", "termine la session", "on arrête là"), or runs /session-end.
argument-hint: [short summary of what was accomplished]
---

# session-end

Close the session opened by `/session-start`. The optional argument is a short summary of the work done; otherwise infer it from the conversation.

Use the same conventions as `session-start` (project root, project name, dev name, tracking files, `SESSION_GLOBAL_TRACKING`, shell date/time).

## Steps

1. **Detect project and dev** (same as `session-start`).

2. **Session number**: take the highest `NN` among `resume/session_NN_*.md` and add 1 (start at 01 if the folder is empty or missing; create it). Do not count files.

3. **Write the resume** `resume/session_<NN>_<YYYYMMDD>_<HHMM>.md`:
   ```markdown
   # Session <NN> Summary - <short title>

   **Date:** <YYYY-MM-DD>
   **Dev:** <dev>
   **Duration:** <computed from the tracking row>

   ## Done
   <bullets grouped by area (e.g. Backend, Frontend, Infra, Docs); omit empty areas>

   ## Technical Decisions
   <notable choices and why, or "None">

   ## Next Steps
   <what remains>
   ```
   Write in English, with bullets rather than paragraphs.

4. **Project summary**: if `docs/PROJECT_SUMMARY.md` exists and the session added modules, entities or architectural changes, update it. Do not create it.

5. **Close project tracking**: find the **last** open row (empty End Time) for this dev in `TIME_TRACKING.md`. Fill:
   - End Time: current time
   - Duration: End − Start, formatted `Xh YYm`. If End is before Start, the session crossed midnight: add 24h.
   - Session Summary: the argument or an inferred summary (about 10 words).

   Then recompute and update the `**Total Time Spent: Xh YYm**` line (sum of all Duration cells), creating it as the last line if missing.

6. **Close global tracking** (only if `SESSION_GLOBAL_TRACKING` is set): same on that file, matching the last open row for this project **and** this dev. No total line is required there.

7. **Local containers**: if the docker MCP tools are available, list the running containers of the project on the local host only (never a remote host such as a NAS), as in `session-start` step 3. If there are any, ASK the user whether to stop them (a question with the container names, default suggestion: stop, to free the machine). Stop them with `docker_stop` only on a yes, application and proxy first, database last. Never stop containers of other projects, and never decide on your own. Skip silently when none are running.

8. **Confirm** with the resume filename, the project's total time and whether the containers were stopped or left running.

If no open row is found, say so and ask whether to create a closed row from a start time given by the user, rather than guessing.
