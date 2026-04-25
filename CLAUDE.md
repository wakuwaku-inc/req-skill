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
- `{{DoD}}` — 完成条件 (依頼者視点 / エンジニア視点に分けたチェックリスト)
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

## Sync rules (/req-brainstorm)

When you add, rename, or remove a flow step in `/req-brainstorm`:

1. Update `skills/req-brainstorm/SKILL.md`.
2. Update `skills/req-brainstorm/SKILL.ja.md` to mirror.
3. Update the `/req-brainstorm` smoke-test checklist (below).
4. Update `README.md` and `README.ja.md` if user-visible behavior changed.

When you add, rename, or remove a placeholder in `templates/requirements.md`:

The existing 5-file sync rule (template ↔ `/req` SKILL.md create-mode ↔ `/req` SKILL.md update-mode U2 anchors ↔ `CLAUDE.md` ↔ `/req` SKILL.ja.md) extends to a 9-file sync — also update:

6. `skills/req-brainstorm/SKILL.md` (Step 4S full-draft section AND Step 4C placeholder ordering).
7. `skills/req-brainstorm/SKILL.ja.md`.
8. `skills/req-lite/SKILL.md` (Placeholders inference table AND Step 3 draft-display block).
9. `skills/req-lite/SKILL.ja.md`.

Drift between these nine files is the dominant failure mode for placeholder changes. Do not merge changes that touch only a subset.

## Sync rules (/req-lite)

When you add, rename, or remove a flow step in `/req-lite`:

1. Update `skills/req-lite/SKILL.md`.
2. Update `skills/req-lite/SKILL.ja.md` to mirror.
3. Update the `/req-lite` smoke-test checklist (below).
4. Update `README.md` and `README.ja.md` if user-visible behavior changed.

When you change Hard Rules / Step 0.5 prompt / Step 0.6 wording / Do Not in `/req-lite`:

See "Structural alignment policy" below — `/req` is canonical. Changes to these sections must be applied across `/req`, `/req-brainstorm`, and `/req-lite` together.

## Structural alignment policy (`/req` is canonical)

Across `/req` / `/req-brainstorm` / `/req-lite` (skills that share Hard Rules, workspace detection, and Do Not lists), `/req` is the canonical source. The following sections must remain text-aligned to `/req`'s wording, except where a per-skill behavioral difference forces divergence (e.g., `/req-brainstorm` has no update mode, so its Hard Rule 3 lacks an update-mode bullet):

- Hard Rules 1–5 (especially Rule 3 output-path wording and Rule 5 placeholder-syntax containment).
- Step 0.5 workspace-detection prompt and abort/skip messages.
- Step 0.6 workspace context-read description (file list, `## 取り込み済み外部コンテンツ` exclusion, prompt-injection warning).
- Do Not section: shared items (fabricate / write-outside / git / placeholder leak / `## 取り込み済み外部コンテンツ` instructions).

Each per-skill SKILL.md must literally inherit the canonical wording for the items above. Allowed adaptations:

1. Replace `/req` with the actual command name (`/req-brainstorm`, `/req-lite`).
2. Append (do not replace) skill-specific Hard Rules and Do Not items at the end of the respective lists.
3. Remove items that do not apply (e.g., `/req-brainstorm` removes the update-mode bullet from Hard Rule 3).

When updating any of these shared sections, update all affected skills in the same change. The 7-file (or 9-file with `/req-lite`) placeholder sync rule above applies; the structural alignment scope is identical.

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
- [ ] `/req` Step 3 sub-step 10 (DoD) produces a proposal with both `**依頼者視点 (受け入れ基準)**` and `**エンジニア視点 (技術完了基準)**` bold-label sections AND includes hierarchical-absorption guidance (「上位の受け入れ基準で吸収できる」 / 「テストで担保できる粒度」).
- [ ] `/req` DoD with one viewpoint empty (e.g. copy-only change) omits that label section entirely; the remaining viewpoint still uses the bold-label format.
- [ ] `/req` output file has no `### **誰がどうやって確認しますか?**` section and no `{{verify}}` prompt anywhere in Step 3.
- [ ] `/req` output `## ✅ 完成条件` section uses bold-label structure (no H3 inside the section).
- [ ] `/req --update` on a pre-migration doc (legacy `### **何ができればOKですか?（チェックリスト）**` + `### **誰がどうやって確認しますか?**`) reverse-maps DoD via the legacy fallback anchor; the verify section is preserved at its original relative position as a custom section in U8 output.
- [ ] `/req --update` U3 summary shows 「完成条件: 依頼者視点N項目 / エンジニア視点M項目」 when role-split structure detected; falls back to flat item count when free-form.

After any /req-setup change:

- [ ] `/req-setup` in an empty repo produces `docs/req-skill/` with `product.md`, `glossary.md`, `decisions.md`, `references.md`, and `requirements/`.
- [ ] `/req-setup` in a repo with existing `docs/requirements/*` proposes extraction candidates and honors 採用 / スキップ / 修正.
- [ ] `/req-setup` second run presents the re-run menu (not scaffold).
- [ ] `/req-setup` with Notion MCP disabled records URL only with an explicit degrade message.
- [ ] `/req-setup` run where the LLM-generated summary still contains a real residual secret pattern after redaction degrades to URL-only with explanation.
- [ ] `/req-setup` run on a Notion page with embedded AWS pre-signed image URLs (`X-Amz-Signature`) produces a summary first; if the summary happens to verbatim-copy the URL, redaction substitutes `[redacted: signed-url]` and notifies 「要約から一時 URL / トークンを除去しました」. Most image-rich pages should produce clean summaries with no redaction triggered.
- [ ] `/req-setup` run where the fetched content contains the bare word `password` in prose (no `=value` suffix) does NOT trigger redaction or URL-only fallback.
- [ ] `/req-setup` invoked from CWD under `/tmp` prompts for a persistent target path.
- [ ] `/req-setup` persistent-mode target collision triggers 上書き / `_2` / 中断.
- [ ] `/req` with `docs/req-skill/product.md` present does not prompt for setup.
- [ ] `/req` without workspace offers `/req-setup` and can be skipped.
- [ ] `/req` skip-setup path still produces a requirements doc (regression).
- [ ] `/req` workspace-aware mode: new term detected in Step 3 triggers glossary-capture prompt.
- [ ] `/req` workspace-aware mode writes the resulting requirements doc to `<CWD>/docs/req-skill/requirements/YYYY-MM-DD_<title>.md`, NOT `<CWD>/docs/requirements/`.
- [ ] `/req` no-workspace mode preserves the legacy path `<CWD>/docs/requirements/YYYY-MM-DD_<title>.md`.
- [ ] User-facing prompts NEVER show `{{placeholder}}` syntax (e.g. `({{requirements}})`); Step 2 draft confirmation, Step 3 sub-step prompts, and U3 current-state summary use Japanese labels only.
- [ ] `/req` end reconciliation presents staged changes and honors 一括承認 / 個別選択 / 全スキップ.
- [ ] `/req-setup` re-run option 3 (regenerate templates) modifies only marker regions.
- [ ] `/req-setup` with no `docs/**` content and no materials falls back to minimal scaffold (α).

After any /req-brainstorm change:

- [ ] `/req-brainstorm ボタンの色を青にしたい` triggers Step 4S (fast-track), produces a single full-draft confirmation listing all 11 fields (post-`{{verify}}`-removal: 課題, 関連URL, 要望, 要件, デザインURL, 参考サイト, その他資料, 希望納期, 納期理由, 完成条件, 補足).
- [ ] `/req-brainstorm 通知機能を追加したい` triggers Step 4C (active-proposal dialogue) with at least one implementation-vs-operational or scope alternative proposed.
- [ ] `/req-brainstorm` without arguments prompts 「何について壁打ちしますか？」.
- [ ] Step 4C: each branching question presents the AI's recommendation with reasoning, not a bare options list.
- [ ] Step 4C decision capture stages a `decisions.md` entry whose body includes rejected alternatives with reasons.
- [ ] Step 4S in workspace-aware mode stages URL fields as reference candidates only (no auto-decision staging for `{{requirements}}` / `{{DoD}}`).
- [ ] Step 4S 「もっとじっくり議論したい」 escalates to Step 4C preserving Step 2 confirmed drafts.
- [ ] Workspace-aware mode writes to `<CWD>/docs/req-skill/requirements/`. No-workspace mode writes to `<CWD>/docs/requirements/`.
- [ ] Existing target path triggers 上書き / `_2` / 中断 prompt.
- [ ] Early termination (「ここまでで」) before Step 6 produces a doc with `{{foo}}` tokens preserved AND discards workspace stage.
- [ ] Output file has YAML frontmatter `title:` and no h1.
- [ ] User-facing prompts NEVER show `{{placeholder}}` syntax.
- [ ] Flow-progress narration NEVER leaks `{{placeholder}}` syntax. Transitional phrases like 「次は X を提案します」 / 「X の中身が固まりました」 must use Japanese labels (「完成条件」, 「要件」 etc.), never `{{DoD}}` / `{{requirements}}` (regression guard for v0.4.1 fix).
- [ ] `/req` (existing) still produces an unchanged output (regression guard — no shared-state leakage from /req-brainstorm changes).
- [ ] Step 9 reconciliation honors 一括承認 / 個別選択 / 全スキップ.
- [ ] `/req-brainstorm` does NOT accept `--update` (the user is told to use `/req --update`).
- [ ] `/req-brainstorm` Step 4S full-draft 完成条件 line shows both 依頼者視点 / エンジニア視点 sub-drafts (or explicit 「特になし (テストで担保)」 for trivially empty viewpoint).
- [ ] `/req-brainstorm` Step 4C does NOT present a verify proposal (former Pattern 5 removed; Pattern 5 is now metadata-fields).
- [ ] `/req-brainstorm` Step 4C Pattern 4 (DoD proposal) presents both `**依頼者視点 (受け入れ基準)**` and `**エンジニア視点 (技術完了基準)**` sections AND includes hierarchical-absorption guidance text.
- [ ] `/req-brainstorm` Step 4C: user saying 「視点で分けなくていい」 falls back to flat single checklist; in workspace-aware mode, this preference is staged in `decisions.md`.
- [ ] `/req-brainstorm` Step 4S 「もっとじっくり議論したい」 escalation to 4C preserves both-viewpoint draft state.

After any /req-lite change:

- [ ] `/req-lite "TOPページの「○○」を「□□」に変更してください"` produces a file at `docs/[req-skill/]requirements/YYYY-MM-DD_<title>.md` with 課題 / 要望・手段 / 要件 populated automatically.
- [ ] `/req-lite` (no args) prompts 「依頼文と添付資料（パス可）を貼ってください」.
- [ ] `/req-lite` with very short `$ARGUMENTS` (e.g., 「お願い」) triggers the same prompt.
- [ ] `/req-lite "..."` with a `.pdf` path in the text reads the PDF via Read and includes its content in inference.
- [ ] `/req-lite "..."` with a `.xlsx` path triggers 「内容をテキストで貼り付けてください」 fallback.
- [ ] `/req-lite "..."` with a customer attribution like 「○○社からの依頼:」 preserves it at the head of 課題.
- [ ] `/req-lite` Step 4 always asks 関連URL even if the request text already contains URLs (the explicit confirmation step is mandatory). When URLs are present, they are presented as candidates rather than the plain ask.
- [ ] `/req-lite` Step 5 always asks 希望納期; sub-asks 納期理由 only when 希望納期 is non-empty. When a date-like string is present, it is presented as a candidate rather than the plain ask.
- [ ] `/req-lite` Step 6 batch question parses URLs into 参考サイト, checklists into 完成条件, file paths into その他資料, prose into 補足.
- [ ] `/req-lite --update <path>` aborts with the redirect message pointing at `/req --update`.
- [ ] `/req-lite` in workspace-aware mode writes to `docs/req-skill/requirements/`; no-workspace mode writes to `docs/requirements/`.
- [ ] `/req-lite` in workspace-aware mode does NOT present a workspace-reconciliation prompt at the end (no staging).
- [ ] `/req-lite` in workspace-aware mode does NOT prompt 「"<term>" を用語集に追加しますか？」 even when a new term is introduced.
- [ ] `/req-lite` in workspace-aware mode does NOT prompt 「この決定を `decisions.md` に追加しますか？」 at confirmation points.
- [ ] `/req-lite` early termination (「ここまでで」) before Step 8 keeps unresolved `{{foo}}` tokens verbatim.
- [ ] Output file has YAML frontmatter `title:` and no h1 at the top.
- [ ] Existing target path triggers 上書き / `_2` / 中断 prompt (same as `/req`).
- [ ] User-facing prompts NEVER show `{{placeholder}}` syntax. Step 3 draft confirmation uses 「課題・困り事・背景」 (not 「{{issues}}」), Step 4 ask uses 「関連URL」 (not 「{{issue_urls}}」), etc.
- [ ] Flow-progress narration NEVER leaks `{{placeholder}}` syntax. Transitional phrases like 「次は X を確認します」 must use Japanese labels (「希望納期」, 「関連URL」 etc.), never `{{due}}` / `{{issue_urls}}` (regression guard for v0.4.1 fix in `/req-brainstorm`).
- [ ] `/req-lite` no-workspace prompt wording matches `/req` canonical: 「ワークスペースが未初期化です。`/req-setup` を実行してから続けますか？ - はい / - スキップ」 with the exact paren-explanation that running はい aborts the current command.
- [ ] `/req` and `/req-brainstorm` continue to behave unchanged after `/req-lite` is added (regression guard — no shared state).
- [ ] Generated `/req-lite` output is consumable by `/req --update <path>` for later expansion.

## Output path contract (/req-setup)

/req-setup writes only inside `<target>/docs/req-skill/`. Never writes `CLAUDE.md`. Never modifies git state.

## Output path contract

Create mode (depends on `/req` start-time workspace detection — see `skills/req/SKILL.md` Step 0.5):
- workspace-aware (workspace at `<CWD>/docs/req-skill/product.md` exists): writes only inside `<CWD>/docs/req-skill/requirements/`.
- no-workspace: writes only inside `<CWD>/docs/requirements/` (legacy).

Update mode: writes to the input file path (file input) OR the Notion page at the input URL (Notion input). On Notion write failure, writes a local fallback — workspace-aware path (`<CWD>/docs/req-skill/requirements/YYYY-MM-DD_<title>.md`) if the workspace exists at fallback time, else legacy path (`<CWD>/docs/requirements/YYYY-MM-DD_<title>.md`).

In all modes: never modify git state, never read files outside `templates/requirements.md` in this plugin (template) or the user-specified input (update mode).

/req-brainstorm mode (workspace-aware vs no-workspace branch decided at Step 0.5):
- workspace-aware: writes only inside `<CWD>/docs/req-skill/requirements/`.
- no-workspace: writes only inside `<CWD>/docs/requirements/` (legacy).
- No update mode (use `/req --update`).

/req-lite mode (workspace-aware vs no-workspace branch decided at Step 0.5):
- workspace-aware: writes only inside `<CWD>/docs/req-skill/requirements/`.
- no-workspace: writes only inside `<CWD>/docs/requirements/` (legacy).
- No update mode (use `/req --update`).
- No workspace staging (no writes to `glossary.md` / `decisions.md` / `references.md`).

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
- Update mode for `/req-brainstorm` (use `/req --update` for existing requirements).
- Persisting brainstorm dialogue transcripts as a separate artifact (decisions go to `decisions.md`; the rest is conversation).
- Auto-handoff from `/req-brainstorm` to `/req` mid-session (output is the requirements file; nothing to hand off).
- Update mode for `/req-lite` (use `/req --update` for existing requirements).
- Workspace staging from `/req-lite` (use `/req` if you want glossary / decisions / references to grow from a request).
- Auto-handoff from `/req-lite` to `/req-brainstorm` or `/req` mid-session (the lite skill suggests the relevant command and exits; the user re-invokes).
