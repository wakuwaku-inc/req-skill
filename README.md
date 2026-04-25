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

## /req-setup — product context workspace

`/req-setup` initializes a per-product context workspace at `docs/req-skill/` that `/req` reads during requirements capture. The workspace contains `product.md` (overview), `glossary.md` (terminology), `decisions.md` (accumulated decisions), `references.md` (external materials index), and `requirements/` (absorbs /req output).

### First run

Run `/req-setup` in your project. The skill scans existing `docs/**/*.md` for term / decision / reference candidates, interviews you for product name / description / stakeholders, and optionally ingests external materials (URLs, Notion pages, Google Drive documents, local folders). All candidates are reviewed per-item before write.

### Re-run

Running `/req-setup` when the workspace already exists presents a menu:
1. Add more references.
2. Rescan `docs/**` for new candidates.
3. Regenerate template marker regions (with diff preview).
4. Abort.

User edits outside `<!-- req-workspace:auto --> ... <!-- /req-workspace:auto -->` markers are never modified.

### Integration with /req

- `/req` detects `docs/req-skill/product.md` at start. If absent, it offers to run `/req-setup` (skippable).
- In workspace-aware mode, `/req` reads workspace headers as background context, prompts to add new terms / decisions to the workspace during the interview, and reconciles changes at the end.
- `/req --update` is unaffected; it neither reads nor writes the workspace.

### Claude Desktop Cowork

When `/req-setup` runs in a Cowork conversation with no folder attached (CWD under `/tmp`, `/var/folders`, or `/private/tmp`), it offers to write to a persistent location (default `$HOME/Documents/req-workspaces/<slug>/`) and instructs you to attach that folder in Cowork for subsequent sessions.

### Security

External materials are summarized and stored in `references.md`. Content matching common secret patterns (API keys, passwords, tokens, bearer) is recorded as URL-only. Ingested summaries are isolated to a dedicated section and treated as background data, not instructions.

## /req-brainstorm — active-proposal brainstorming

`/req-brainstorm [request]` runs an active-proposal dialogue with you before producing a requirements document. Where `/req` is a passive interviewer (asks one field at a time and waits), `/req-brainstorm` proactively proposes 2–3 options with tradeoffs and a recommendation at each branching point — implementation vs. operational workaround, MVP vs. full scope, layout / placement alternatives. Final output is the same `requirements/YYYY-MM-DD_<title>.md` file as `/req`.

### Two branches

**Fast-track (Step 4S)** — for trivial requests (single-screen change, obvious DoD, no real alternatives), the AI drafts ALL fields from your request in one go and asks you to confirm or edit. No 13 sequential prompts.

**Brainstorm dialogue (Step 4C)** — for everything else, the AI walks you through `{{requirements}}` → `{{DoD}}` → `{{verify}}` → metadata, proposing alternatives with reasoning at each branch. Decisions made during the dialogue (chosen + rejected options + reasons) are staged into the workspace's `decisions.md` for future reference.

The branch is decided automatically by the AI based on request complexity. You can override at any time: 「じっくりやりたい」 forces dialogue mode; 「シンプルでいい」 forces fast-track.

### When to use which

| Situation | Command |
| --- | --- |
| Requirements already clear, just want to formalize | `/req` |
| Want to think through implementation/operational tradeoffs | `/req-brainstorm` |
| Modifying an existing requirements doc | `/req --update` |

### Workspace integration

Same as `/req`: in workspace-aware mode (`docs/req-skill/product.md` exists), `/req-brainstorm` reads workspace headers as background context, prompts to add new terms / decisions / references during the dialogue, and reconciles changes after the requirements file is written.

### No update mode

`/req-brainstorm` does not have an `--update` mode. To modify an existing requirements doc, use `/req --update <path-or-notion-url> [what]`.

## Contributing

See `CLAUDE.md` for placeholder list, sync rules between SKILL.md and the template, and the smoke-test checklist.

## License

MIT
