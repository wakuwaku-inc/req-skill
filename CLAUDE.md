# req-skill — Maintenance Notes

This file captures information that future contributors (human or Claude) need to maintain the req-skill plugin correctly. It is loaded automatically by Claude Code when working inside this repository.

## What this repo is

A Claude Code plugin that provides a single skill, `/req [what]`, for interactive development-requirements capture. When run in a user's project, the skill writes a filled requirements document to `<CWD>/docs/requirements/YYYY-MM-DD_<title>.md`.

## Source of truth: placeholders

The canonical placeholder list lives in `templates/requirements.md`. The skill prompt `skills/req/SKILL.md` must reference exactly these placeholders and no others:

- `{{title}}` — Notion page title (YAML frontmatter only; never emitted as h1).
- `{{issues}}` — 課題・困り事・背景
- `{{issue_urls}}` — 関連URL
- `{{requests}}` — 要望・手段
- `{{requirements}}` — 要件
- `{{designs}}` — デザインURL
- `{{references}}` — 参考サイト
- `{{others}}` — その他資料
- `{{due}}` — 希望納期
- `{{due_reasons}}` — 納期理由
- `{{DoD}}` — 完成条件
- `{{verify}}` — 確認方法
- `{{notes}}` — その他・補足

## Sync rules (SKILL.md ↔ templates/requirements.md ↔ Update Mode)

When you add, rename, or remove a placeholder:

1. Update `templates/requirements.md`.
2. Update the placeholder table and the relevant question step in `skills/req/SKILL.md`.
3. Update the Update Mode anchor table (in SKILL.md `### Update Step U2`) — the reverse-map must stay in sync with the template's heading structure.
4. Update this `CLAUDE.md` list.
5. Update `skills/req/SKILL.ja.md` to mirror the English changes (both Create Mode Step 3 and Update Mode U2).
6. Run the smoke-test checklist below.

Drift between these five files (template, SKILL.md create-mode, SKILL.md update-mode U2 anchors, CLAUDE.md, SKILL.ja.md) is the dominant failure mode for this plugin. Do not merge changes that touch only a subset.

## Sync rules (/req-setup — workspace templates)

When you add, rename, or remove a workspace template file under `templates/workspace/`:

1. Update the template file.
2. Update `skills/req-setup/SKILL.md` Step 5 (Atomic write) placeholder-substitution table.
3. Update `skills/req-setup/SKILL.ja.md` to mirror.
4. Update this `CLAUDE.md` workspace-template list (below).
5. Update the /req-setup smoke-test checklist (below).

The four workspace templates are:

- `templates/workspace/product.md` — product overview, stakeholders.
- `templates/workspace/glossary.md` — glossary, 1 entry per line.
- `templates/workspace/decisions.md` — decisions, summary-first.
- `templates/workspace/references.md` — references index, includes `## 取り込み済み外部コンテンツ` injection-containment section.

Marker convention: every auto-managed region is wrapped in `<!-- req-workspace:auto --> ... <!-- /req-workspace:auto -->`. `/req-setup` re-generation writes only inside these markers. User edits outside markers are preserved.

## Smoke-test checklist (manual)

After any change:

- [ ] `/req ダッシュボード新規画面` produces a file at `docs/requirements/YYYY-MM-DD_<title>.md`.
- [ ] `/req` without arguments prompts 「何について要件を作りますか？」.
- [ ] An existing target path triggers the overwrite / `_2` / abort prompt.
- [ ] An explicitly-skipped optional field ("特になし") results in the placeholder line being deleted while the surrounding heading remains.
- [ ] Early termination keeps unresolved `{{foo}}` tokens verbatim.
- [ ] The output file has YAML frontmatter `title:` and no h1 at the top.
- [ ] `/req --update <path-to-existing-req>.md 要件を1項目追加` updates the file in place; filename unchanged.
- [ ] `/req --update <notion-url>` with Notion MCP active updates the page.
- [ ] `/req --update <notion-url>` when MCP write fails creates a local fallback at `docs/requirements/YYYY-MM-DD_<title>.md`.
- [ ] `/req --update` with no further argument aborts with the missing-requirements message.
- [ ] `/req --update <garbage-string>` aborts with the "path or Notion URL" message.
- [ ] `/req --update <unrelated-md>.md` aborts with the "not a /req requirements doc" message.
- [ ] `/req ダッシュボード新規画面` (no `--update`) still runs create mode unchanged (regression guard).

After any /req-setup change:

- [ ] `/req-setup` in an empty repo produces `docs/req-skill/` with `product.md`, `glossary.md`, `decisions.md`, `references.md`, and `requirements/`.
- [ ] `/req-setup` in a repo with existing `docs/requirements/*` proposes extraction candidates and honors 採用 / スキップ / 修正.
- [ ] `/req-setup` second run presents the re-run menu (not scaffold).
- [ ] `/req-setup` with Notion MCP disabled records URL only with an explicit degrade message.
- [ ] `/req-setup` run where fetched content contains a real residual secret pattern (after sanitization) records URL only and informs the user.
- [ ] `/req-setup` run on a Notion page with embedded AWS pre-signed image URLs (`X-Amz-Signature`) sanitizes the URLs to `[redacted: signed-url]`, generates a summary from sanitized text, and notifies 「画像 URL や一時トークンを除去して要約しました」.
- [ ] `/req-setup` invoked from CWD under `/tmp` prompts for a persistent target path.
- [ ] `/req-setup` persistent-mode target collision triggers 上書き / `_2` / 中断.
- [ ] `/req` with `docs/req-skill/product.md` present does not prompt for setup.
- [ ] `/req` without workspace offers `/req-setup` and can be skipped.
- [ ] `/req` skip-setup path still produces a requirements doc (regression).
- [ ] `/req` workspace-aware mode: new term detected in Step 3 triggers glossary-capture prompt.
- [ ] `/req` end reconciliation presents staged changes and honors 一括承認 / 個別選択 / 全スキップ.
- [ ] `/req-setup` re-run option 3 (regenerate templates) modifies only marker regions.
- [ ] `/req-setup` with no `docs/**` content and no materials falls back to minimal scaffold (α).

## Output path contract (/req-setup)

/req-setup writes only inside `<target>/docs/req-skill/`. Never writes `CLAUDE.md`. Never modifies git state.

## Output path contract

Create mode: writes only inside `<CWD>/docs/requirements/`.

Update mode: writes to the input file path (file input) OR the Notion page at the input URL (Notion input). On Notion write failure, writes a local fallback to `<CWD>/docs/requirements/YYYY-MM-DD_<title>.md`.

In all modes: never modify git state, never read files outside `templates/requirements.md` in this plugin (template) or the user-specified input (update mode).

## Publication notes

- Distribution: GitHub public repo. The repo itself acts as a Claude Code marketplace via `.claude-plugin/marketplace.json`.
- Plugin manifest lives at `.claude-plugin/plugin.json` (NOT repo root — Claude Code only looks under `.claude-plugin/`).
- Marketplace name is declared explicitly as `req-skill` in `marketplace.json`; it is **not** auto-generated from the GitHub `owner-repo` slug. The install command is `/plugin install req-skill@req-skill`.
- When bumping the plugin version, update `.claude-plugin/plugin.json` AND tag a matching `vX.Y.Z` release on GitHub.
- Prompt language: English (`skills/req/SKILL.md`). User-facing output: Japanese.
- Japanese translation of SKILL.md lives at `skills/req/SKILL.ja.md` for documentation only; Claude Code does not auto-load it.

## Known non-goals (deferred)

- Persistent draft across separate Claude sessions.
- Environment-variable-based output directory customization.
- Automatic `git add` / `git commit` of generated files.
- Direct Notion push.
- Monorepo / multi-product support within a single CWD (workaround: cd to subdirectory).
- Workspace compaction / digest mode (v2).
- Programmatic Cowork folder attachment (pending API availability).
