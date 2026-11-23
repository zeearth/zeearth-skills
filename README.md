# zeearth-skills

A [Claude Code](https://claude.com/claude-code) plugin marketplace with small, focused plugins for day-to-day development workflows.

## Install

Prerequisites: [Claude Code](https://claude.com/claude-code) and `git`. `python3` is needed by `conventions-guard`, `docker-npm` (guard hook) and `personas`. `docker-npm` also needs Docker with Compose in the projects where you use it.

1. Add the marketplace (once). Run these inside Claude Code:

   ```
   /plugin marketplace add zeearth/zeearth-skills
   ```

   A full Git URL (`https://github.com/zeearth/zeearth-skills.git`) or a local clone path also works.

2. Install the plugins you want. The part after `@` is the marketplace name, `zeearth-skills`:

   ```
   /plugin install conventions-guard@zeearth-skills
   /plugin install docker-npm@zeearth-skills
   /plugin install personas@zeearth-skills
   /plugin install session-tracking@zeearth-skills
   ```

   The install prompt lets you choose the scope: user (all your projects), project (shared through the repo's `.claude/settings.json`) or local (this project, just for you).

3. Run `/reload-plugins`, or restart Claude Code, so the new skills and hooks load. Check with `/plugin`.

Update and remove:

```
/plugin marketplace update zeearth-skills
/plugin update <plugin>@zeearth-skills
/plugin uninstall <plugin>@zeearth-skills
```

Commands are available as `/session-start`, `/persona`, `/docker-npm`. If a name clashes with another plugin, use the namespaced form, for example `/personas:persona`.

## Plugins

| Plugin | What it does |
| :--- | :--- |
| [docker-npm](plugins/docker-npm) | Install, update, remove, list and audit npm packages inside your project's Docker container, never on the host. |
| [session-tracking](plugins/session-tracking) | `/session-start` and `/session-end`: load project context, write a per-session resume, and keep time-tracking tables up to date. |
| [conventions-guard](plugins/conventions-guard) | Hooks that check every write: braces on control statements, 500-line limit on Node/TS files, no secrets in committed files. |
| [personas](plugins/personas) | `/persona <id>`: expert personas (CTO, web designer, full-stack, DBA, docs, product owner, security); global files are completed, not replaced, per project. |

## License

MIT
