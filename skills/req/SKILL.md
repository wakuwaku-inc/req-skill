---
name: req
description: Use when the user wants to create a development-requirements document interactively, or when the user invokes /req (with or without arguments). Activates on natural-language requests like "draft a requirements doc" in addition to the slash command.
---

# /req — Interactive Development Requirements

**Announce at start:** "Using /req to interview you and produce a requirements document at `docs/requirements/YYYY-MM-DD_<title>.md`."

## Inputs

- `$ARGUMENTS` — the `[what]` text the user typed after `/req`. May be empty.
- `templates/requirements.md` inside this plugin — the source-of-truth template. Load it with Read at the start.

## Hard Rules

1. **Never fabricate answers.** If a placeholder value is not explicitly provided or derivable from the user's own words, you MUST ask.
2. **Output language is Japanese.** Ask and answer in Japanese with the user. Reasoning may be internal English.
3. **Output path** is `<CWD>/docs/requirements/YYYY-MM-DD_<title>.md`. Create `docs/requirements/` if missing. Do not write outside this directory.
4. **Never auto-commit.** Writing the file is enough; the user decides whether to `git add` / `git commit`.

## Placeholders (sync with templates/requirements.md)

| Placeholder | Source question |
| --- | --- |
| `{{title}}` | Auto-generated from content at the end; user confirms. |
| `{{issues}}` | 💡 なぜ必要? — 課題・困り事・背景 |
| `{{issue_urls}}` | 🔗 関連URL (Slack / Chatwork / チケット等) |
| `{{requests}}` | 🎯 何を作りたい? — 要望・手段 |
| `{{requirements}}` | 📱 要件 — 画面 / 機能 / 項目 / 動作 |
| `{{designs}}` | 🎨 デザインURL |
| `{{references}}` | 参考サイト |
| `{{others}}` | その他資料 |
| `{{due}}` | 希望納期 |
| `{{due_reasons}}` | この日までに必要な理由 |
| `{{DoD}}` | ✅ 何ができればOK? (チェックリスト) |
| `{{verify}}` | 誰がどうやって確認するか |
| `{{notes}}` | その他・補足 |

## Flow

### Step 0 — Argument dispatch

Parse `$ARGUMENTS`:

1. Split on whitespace. If the first token is `--update`, enter **update mode**:
   - The second token is `<requirements>` (local file path or Notion URL).
   - Everything after the second token (whitespace preserved) is `[what]`. May be empty.
   - If `<requirements>` is missing, abort: 「対象の要件書を指定してください (ファイルパス or Notion URL)」
   - Proceed to the **Update Flow** (section "Update Mode" below). Do NOT run Steps 1–8 of create mode.
2. Otherwise, run **create mode**:
   - If `$ARGUMENTS` is empty, ask: 「何について要件を作りますか？」and wait for input.
   - Proceed to Step 1.

### Step 1 — Load template
Read `templates/requirements.md` from this plugin. If missing, tell the user to reinstall the plugin and stop.

### Step 2 — Initial draft (hybrid mode)
From `$ARGUMENTS`, draft an initial draft of `{{issues}}` and `{{requests}}`. Present to the user as:

「以下の理解で合っていますか？修正点があれば教えてください。
- 課題・困り事・背景 (`{{issues}}`): <draft>
- 要望・手段 (`{{requests}}`): <draft>」

Wait for confirmation or correction. Update drafts.

### Step 3 — Sequential deep-dive (ask only for empty fields)
In this exact order:

1. `{{issues}}` — confirm / tighten from Step 2 draft.
2. `{{issue_urls}}` — 「課題・困り事・背景に関連するURLはありますか？ Slack / Chatwork / チケット等、複数ある場合は全て教えてください。」 Collect all URLs.
3. `{{requests}}` — confirm / tighten from Step 2 draft.
4. `{{requirements}}` — deep-dive the requirements. Probe missing information thoroughly. If the user does not know, explicitly say: 「不明点があれば持ち帰って調べてから戻ってきてください。続きから再開できます。」 Once back with info, propose a structured `{{requirements}}` (screens / fields / behaviors) and confirm.
5. `{{designs}}` — 「デザインに関するURLはありますか？ 複数ある場合は全て教えてください。」
6. `{{references}}` — 「参考サイトはありますか？ 複数ある場合は全て教えてください。」
7. `{{others}}` — 「その他の資料・参考情報はありますか？」
8. `{{due}}` — 「希望納期はありますか？」
9. `{{due_reasons}}` — if `{{due}}` present: 「なぜその日までに必要ですか？ 軽微なら『特になし』で構いません。」
10. `{{DoD}}` — propose a DoD checklist derived from the requirements and ask the user to confirm / edit. Ask in the pattern: 「以下が完成条件で合っていますか？ 追加・修正あれば教えてください。」
11. `{{verify}}` — 「完成の確認は誰がどうやって行いますか？」
12. `{{notes}}` — 「その他・補足はありますか？」

### Step 4 — Title generation
Propose 2-3 title candidates based on the collected content. Example:
「タイトル候補:
1. <candidate A>
2. <candidate B>
3. <candidate C>
どれがよいですか？ (独自案もOK)」

Wait for selection. Store as `{{title}}`.

### Step 5 — Placeholder cleanup
Take the loaded template and substitute every `{{placeholder}}` with the captured value. For any placeholder the user did not supply (including explicit "特になし" with no content), remove the single placeholder line but keep the surrounding heading.

### Step 6 — Compute output path and confirm
- `DATE` = today (JST if available, else local) in `YYYY-MM-DD`.
- `TITLE` = the user-confirmed title with ASCII spaces replaced by `-`. Japanese characters are preserved.
- `TARGET` = `<CWD>/docs/requirements/{DATE}_{TITLE}.md`.

If `TARGET` exists, ask:
「同名ファイルが既に存在します: `{TARGET}`
- 上書き
- `_2` サフィックスを付けて別名で保存
- 中断
どうしますか？」

Apply the choice. For `_2`, increment until a free name is found.

Tell the user the resolved path and ask for final confirmation before writing.

### Step 7 — Write file
Use Write with the filled content. Create `<CWD>/docs/requirements/` if missing. On write error, show the error and offer to retry or choose a different path.

### Step 8 — Report completion
Show the absolute path written. Do not run `git add` or `git commit`.

## Early termination
If the user says "ここまでで" / "もういい" before Step 5, proceed to Step 5 with whatever is filled. Keep unresolved `{{foo}}` tokens verbatim in the output so the user can finish by hand. Do NOT delete them in this case.

## Do Not
- Do not fabricate URLs, dates, or requirement details.
- Do not write outside `<CWD>/docs/requirements/`.
- Do not auto-commit or touch git.
- Do not load or modify any other plugin file besides `templates/requirements.md` during execution.
