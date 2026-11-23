# conventions-guard

Three [Claude Code](https://claude.com/claude-code) hooks that turn written conventions into checks Claude cannot forget. They run on every `Write` and `Edit`, and a failure is sent back to Claude so it fixes the code in the same turn. Meant for developers who keep a conventions file (such as `CLAUDE.md`) and want it enforced, not just read.

## What it checks

| Hook | When | Rule |
| :--- | :--- | :--- |
| `secret-guard.py` | before the write | Refuses known credential formats (AWS, GitHub, `sk-`, Slack, Google API keys, private keys) in any file except git-ignored `.env*`. In `docker-compose*.yml`, a variable whose name ends in `PASSWORD`, `PASSWD`, `SECRET`, `TOKEN`, `API_KEY`, `PRIVATE_KEY` or `ENCRYPTION_KEY` must be `${VAR}`; in `.env.example` it must be empty. |
| `code-guard.py` | after the write | No `if` / `for` / `while` / `else` without braces in `.ts .tsx .js .jsx .mjs .cjs` (only the lines just written are checked, template literals are skipped). In a project with a `package.json`, a file over 500 lines must be reduced by extracting reusable functions into a shared package. |
| `lang-guard.py` | after the write | Markdown files (`.md`, `.mdx`) must be in English. A stop-word count flags French prose in the text just written (code blocks ignored). To record a file that is French on purpose, put `<!-- lang: fr -->` in its first 5 lines. `.claude/` and `CHANGELOG*` are skipped. |

`code-guard.py` ignores `node_modules`, `.next`, `dist`, `build`, `coverage`, `drizzle/meta`, `*.d.ts` and `.claude/`. The secret check has no ignore list.

## Install

```
/plugin marketplace add zeearth/zeearth-skills
/plugin install conventions-guard@zeearth-skills
```

Then run `/reload-plugins` or restart Claude Code. Requires `python3` on the host. Files written through `Bash` (heredocs) are not seen: the hooks watch `Write` and `Edit` only.

## Example

Claude writes `if (user) return user;` in a `.ts` file. The hook exits with an error listing the offending line, and Claude rewrites it with braces in the same turn. To bypass a check, disable the plugin (`/plugin`) or edit the constants below.

## Tuning

Edit the plugin files in place (changes are lost on update, or fork the repo). The limits live at the top of `hooks/code-guard.py` (`MAX_LINES`, `CODE_EXTENSIONS`, `IGNORED_PARTS`), `hooks/lang-guard.py` (`FRENCH_RATIO`, `MIN_WORDS`) and `hooks/secret-guard.py` (`TOKEN_PATTERNS`, `SECRET_NAME`).

## License

MIT
