# req-skill

A Claude Code plugin that adds a `/req [what]` skill for interactive development-requirements capture. The skill interviews the requester, fills a template, and writes the final document to `docs/requirements/YYYY-MM-DD_<title>.md` in your project.

## Why

Development-request tickets often arrive vague. This skill enforces a structured interview (issues → requests → requirements → designs → schedule → DoD → verification) so implementers get a complete brief and requesters don't forget fields.

## Install

Clone this repo into your Claude Code plugins directory:

```bash
git clone https://github.com/wkwk/req-skill ~/.claude/plugins/req-skill
```

Or add it as a local plugin following your Claude Code configuration conventions.

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
