# req-skill

A Claude Code plugin that adds a `/req [what]` skill for interactive development-requirements capture. The skill interviews the requester, fills a template, and writes the final document to `docs/requirements/YYYY-MM-DD_<title>.md` in your project.

## Why

Development-request tickets often arrive vague. This skill enforces a structured interview (issues → requests → requirements → designs → schedule → DoD → verification) so implementers get a complete brief and requesters don't forget fields.

## Install

In Claude Code, register this repo as a plugin marketplace and install the plugin:

```
/plugin marketplace add https://github.com/wakuwaku-inc/req-skill
/plugin install req-skill@req-skill
```

The marketplace name `req-skill` is declared in `.claude-plugin/marketplace.json`.

## Usage

In any project directory:

```
/req ダッシュボードに新規画面を追加したい
```

Or without arguments:

```
/req
```

The skill will ask follow-up questions in Japanese and save the completed requirements document to `docs/requirements/YYYY-MM-DD_<title>.md`.

### Updating an existing requirements doc

Use `--update` to diff-edit an existing requirements document instead of creating a new one:

```
/req --update <requirements>            # prompts for the change interactively
/req --update <requirements> [what]     # [what] is the change summary (e.g. "要件を1項目追加")
```

`<requirements>` may be:

- A local file path (absolute or repo-relative) — Claude rewrites the file in place. The filename does not change even if the title changes.
- A Notion page URL — Claude fetches the page via the Notion MCP server, applies the change, and writes it back to the same page. If the Notion write fails (missing permissions, network), Claude falls back to writing a local copy at `docs/requirements/YYYY-MM-DD_<title>.md`.

The update flow only deep-dives the fields you say you want to change. It merges by AI judgment (replace for corrections, append for additions) and asks you when the intent is ambiguous.

Notion URL support requires the Notion MCP server configured and authenticated in your Claude Code environment.

## Output format

- YAML frontmatter with `title:` (for Notion compatibility).
- Headings in Japanese with emoji.
- Placeholders (`{{foo}}`) in `templates/requirements.md` are substituted with user-provided content. Unanswered optional fields are removed but headings remain.

## Layout

- `skills/req/SKILL.md` — skill prompt (English, auto-loaded).
- `skills/req/SKILL.ja.md` — Japanese documentation translation.
- `templates/requirements.md` — requirements template.
- `CLAUDE.md` — maintenance notes for contributors.
- `README.ja.md` — Japanese README.

## Contributing

See `CLAUDE.md` for placeholder list, sync rules between SKILL.md and the template, and the smoke-test checklist.

## License

MIT
