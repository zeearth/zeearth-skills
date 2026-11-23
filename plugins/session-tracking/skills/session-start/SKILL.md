---
name: session-start
description: Load project context and open a work session in the time-tracking file(s). Use when the user asks to start, begin or open a work session (e.g. "start the session", "débute la session", "on commence"), or runs /session-start.
---

# session-start

Load the project context and open a session row in the time-tracking file(s). Pair with `/session-end`.

## Conventions (shared with session-end)

- **Project root**: `git rev-parse --show-toplevel`, falling back to the current directory. Never use a subfolder.
- **Project name**: the root folder's basename (or the title of `CLAUDE.md` if present).
- **Dev name**: `git config user.name`, then `git config --global user.name`; if both are empty, ask the user.
- **Project tracking file**: `<root>/TIME_TRACKING.md`.
- **Global tracking file** (optional): the path in the `SESSION_GLOBAL_TRACKING` environment variable (check with `printenv SESSION_GLOBAL_TRACKING`). If unset or empty, skip every "global" step silently.
- **Resume folder**: `<root>/resume/`.
- Date and time come from the shell (`date +%F`, `date +%H:%M`), never from memory.

## Steps

1. **Detect project and dev** with the conventions above.

2. **Load context**:
   - Read `docs/PROJECT_SUMMARY.md` if it exists, otherwise `CLAUDE.md`, otherwise `README.md`.
   - Read the most recent file in `resume/` (highest filename), if any.
   - Tell the user the current state in 2–3 sentences.

3. **Start the local containers** (skip silently when the docker MCP tools are unavailable; a project is expected to have a `docker-compose.yml` / `compose.yml` at its root, so if none exists just say so):
   - Read the compose file's service names to know what belongs to the project (ignore profiles and one-shot services such as `migrate`, `tools`, `init`, `seed`).
   - Load the MCP tools if they are deferred (`ToolSearch` with `select:mcp__docker__docker_list_hosts,mcp__docker__docker_list_containers,mcp__docker__docker_start`), call `docker_list_hosts` and use the local host only (never a remote host such as a NAS).
   - Call `docker_list_containers` with all containers included and keep those whose compose label `com.docker.compose.project` equals the project name (or whose name starts with `<project>-`).
   - Start every stopped one with `docker_start`, dependencies first (database, then app, then proxy). Containers are not set to start at boot, so this is the normal way they come up.
   - If the services are missing (never created, or removed), the MCP cannot create them: run the project's own command, `make up` if the Makefile has it, otherwise `docker compose up -d`, from the project root.
   - If something still does not start, report it instead of guessing. Mention in the confirmation what was started or already running.

4. **Open project tracking**: insert a row after the last table row of `TIME_TRACKING.md`. If a `**Total Time Spent: ...**` line closes the file, keep it last.
   ```
   | <YYYY-MM-DD> | <dev> | <HH:MM> | | | <placeholder, filled at session end> |
   ```
   If the file does not exist, create it:
   ```
   # Time Tracking - <project name>

   | Date | Dev | Start Time | End Time | Duration | Session Summary |
   | :--- | :--- | :--- | :--- | :--- | :--- |
   ```

5. **Open global tracking** (only if `SESSION_GLOBAL_TRACKING` is set): same operation on that file, with the row:
   ```
   | <YYYY-MM-DD> | <project name> | <dev> | <HH:MM> | | | <placeholder> |
   ```
   If the file does not exist, create it (and its folder) with:
   ```
   # Time Tracking — All Projects

   | Date | Project | Dev | Start Time | End Time | Duration | Session Summary |
   | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
   ```

6. **Confirm**: context loaded, local containers up, tracking file(s) updated, ready to work.

If a row for the same dev is already open (no End Time) from a previous day, warn the user instead of adding a second one, and let them decide.
