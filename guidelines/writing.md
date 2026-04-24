---
domain: writing
version: 1
last_updated: 2026-04-24
---

# Writing Guidelines

Applies to all `.md` files in this plugin: `skills/req/SKILL.md`, `skills/req/SKILL.ja.md`, `CLAUDE.md`, `README.md`, `README.ja.md`, `templates/requirements.md`, and spec/plan files under `docs/team-dd/`.

## Language Split

- **English:** prompt text loaded by Claude Code (`skills/req/SKILL.md`), maintainer docs (`CLAUDE.md`), English README, English specs/plans.
- **Japanese:** user-facing output produced by the skill at runtime, Japanese README, `SKILL.ja.md` documentation.
- Translation docs (e.g. `*.ja.md`) are additive siblings of the English original, never replacements.

## Tone & Voice

| Audience | File(s) | Register |
| --- | --- | --- |
| Claude Code (runtime) | `SKILL.md` | Directive, imperative ("Read template", "Ask: …"). Rules numbered when order matters. |
| Maintainers | `CLAUDE.md` | Normative, terse. Sync rules and smoke-tests as checklists. |
| End users | `README.md` / `README.ja.md` | Conversational-instructional. Example blocks + 1-2 sentences per section. |
| Designers / Contributors | `docs/team-dd/` | Token-economy: tables > prose, no filler transitions, no restated rationale. |

## Terminology

| Term | Use | Avoid |
| --- | --- | --- |
| requirements / 要件 | Always for 要件定義 | "spec" (reserved for design docs) |
| create mode / update mode | The two `/req` modes | "new mode" / "diff mode" in prose; OK in spec rationale |
| placeholder | `{{foo}}` token in template | "variable", "slot" |
| Notion MCP | The MCP server for Notion | "Notion API", "Notion plugin" |
| requirements doc | A file produced by `/req` | "spec file", "output" |

Japanese equivalents:

| 日本語 | 用途 |
| --- | --- |
| 要件 / 要件書 | requirements / a requirements doc |
| 作成モード / 更新モード | create mode / update mode |
| プレースホルダ | placeholder |
| 書き戻し | write-back (update mode only) |

## Placeholder Contract

The canonical `{{foo}}` list lives in `templates/requirements.md`. Every `.md` file that mentions the list (`SKILL.md`, `SKILL.ja.md`, `CLAUDE.md`) MUST list exactly the same placeholders — no additions, no omissions. A change in the template requires synchronized edits to all three. See `CLAUDE.md` "Sync rules" for the authoritative list.

## File Pairing

Keep the following pairs synchronized — any change to the English side requires an equivalent change to the Japanese side in the same commit:

- `skills/req/SKILL.md` ↔ `skills/req/SKILL.ja.md`
- `README.md` ↔ `README.ja.md`

`SKILL.ja.md` is documentation only — Claude Code auto-loads only `SKILL.md`. Do not add `name:` frontmatter to `SKILL.ja.md`.

## User-Facing Strings

All user-facing Japanese prompts (e.g. 「何について要件を作りますか？」) are literal strings. They appear identically in both `SKILL.md` and `SKILL.ja.md` — do not translate them to English in `SKILL.md`. Treat them as fixed tokens the skill emits at runtime.

## Structure Conventions

- Use `## Section`, `### Subsection` (ATX). Do not use setext (`===`, `---`) for headings.
- Emoji in headings appears only in `templates/requirements.md` and in user-facing summaries derived from it. Do not add emoji to SKILL / README / CLAUDE / docs headings.
- Tables for enumerations ≥ 3 rows; bullet lists for shorter enumerations.
- Fenced code blocks: use language hints (` ```markdown `, ` ```bash `, ` ```json `) to enable downstream syntax highlighting.
- No trailing whitespace. Single trailing newline at EOF.

## Do Not

- Do not write multi-paragraph rationale in `SKILL.md`. Rationale belongs in specs under `docs/team-dd/specs/`.
- Do not add placeholder rows for future features (YAGNI).
- Do not introduce "usage examples" that diverge from actual skill behavior.
- Do not auto-generate marketing copy (e.g. "Boost your productivity!") in README files.
