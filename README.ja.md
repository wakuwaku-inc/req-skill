# req-skill

開発依頼の要件定義を対話で作成する Claude Code プラグインです。`/req [what]` で起動すると、依頼者に構造化されたヒアリングを行い、`docs/requirements/YYYY-MM-DD_<title>.md` に要件定義書を書き出します。

## 課題感

開発依頼は情報が曖昧なまま来ることが多く、実装者が読み直しや確認で工数を溶かす原因になります。このスキルは「課題→要望→要件→デザイン→スケジュール→DoD→確認方法」の順で構造化ヒアリングを強制し、依頼者の抜け漏れと実装者の確認往復を減らします。

## インストール

Claude Code 上で、このリポジトリを plugin marketplace として登録し、プラグインをインストール:

```
/plugin marketplace add https://github.com/wakuwaku-inc/req-skill
/plugin install req-skill@req-skill
```

marketplace 名 `req-skill` は `.claude-plugin/marketplace.json` で宣言されています。

## 使い方

任意のプロジェクトディレクトリで:

```
/req ダッシュボードに新規画面を追加したい
```

引数なしでも起動可能:

```
/req
```

スキルが日本語で順次質問し、`docs/requirements/YYYY-MM-DD_<title>.md` に要件定義書を書き出します。

### 既存要件書を更新する

`--update` を使うと、新規作成の代わりに既存の要件定義書を差分編集できます:

```
/req --update <requirements>            # 変更内容を対話形式で入力
/req --update <requirements> [what]     # [what] に変更概要を指定 (例: "要件を1項目追加")
```

`<requirements>` に指定できるもの:

- ローカルファイルパス (絶対パスまたはリポジトリ相対パス) — Claude がファイルをその場で書き換えます。タイトルを変更してもファイル名は変わりません。
- Notion ページ URL — Claude が Notion MCP サーバー経由でページを取得し、変更を適用して同じページに書き戻します。Notion 書き戻しに失敗した場合 (権限不足・ネットワークエラー) は、`docs/requirements/YYYY-MM-DD_<title>.md` にローカルコピーを保存します。

更新フローでは、変更すると宣言したフィールドだけを深掘りします。AI の判断でマージし (修正は置換、追加は追記)、意図が曖昧な場合はユーザーに確認します。

Notion URL のサポートには、Claude Code 環境で Notion MCP サーバーの設定と認証が必要です。

## 生成物の形式

- `title:` を含む YAML frontmatter (Notion 連携用)。
- 絵文字付きの日本語見出し。
- `templates/requirements.md` の `{{foo}}` プレースホルダがユーザー入力で置換されます。任意項目で回答が無かったものはプレースホルダ行のみ削除、見出しは残ります。

## 構成

- `skills/req/SKILL.md` — スキルプロンプト本体 (英語、Claude Code が自動ロード)。
- `skills/req/SKILL.ja.md` — 日本語翻訳ドキュメント。
- `templates/requirements.md` — 要件定義テンプレート。
- `CLAUDE.md` — コントリビュータ向けメンテナンス情報。
- `README.md` — 英語版 README。

## /req-setup — プロダクトコンテキストワークスペース

`/req-setup` は `/req` が要件定義ヒアリング中に参照するプロダクトコンテキストワークスペースを `docs/req-skill/` に初期化します。ワークスペースには `product.md`（概要）、`glossary.md`（用語集）、`decisions.md`（決定事項）、`references.md`（外部資料インデックス）、および `requirements/`（/req の出力を格納）が含まれます。

### 初回実行

プロジェクト内で `/req-setup` を実行します。スキルは既存の `docs/**/*.md` から用語・決定事項・参考資料の候補をスキャンし、プロダクト名・説明・ステークホルダーをヒアリングし、外部資料（URL、Notion ページ、Google Drive ドキュメント、ローカルフォルダ）をオプションで取り込みます。すべての候補は書き込み前に1件ずつレビューされます。

### 再実行

ワークスペースが既に存在する状態で `/req-setup` を実行すると、以下のメニューが表示されます:
1. 参考資料を追加する。
2. `docs/**` を再スキャンして新しい候補を抽出する。
3. テンプレートのマーカー領域を再生成する（差分プレビューあり）。
4. 中断する。

`<!-- req-workspace:auto --> ... <!-- /req-workspace:auto -->` マーカー外のユーザー編集は変更されません。

### /req との統合

- `/req` は起動時に `docs/req-skill/product.md` の有無を検出します。存在しない場合は `/req-setup` の実行を提案します（スキップ可能）。
- ワークスペース連携モードでは、`/req` はワークスペースのヘッダーを背景情報として読み込み、ヒアリング中に新しい用語・決定事項をワークスペースに追加するよう促し、終了時に変更を統合します。
- `/req --update` は影響を受けません。ワークスペースの読み書きは行いません。

### Claude Desktop Cowork

`/req-setup` がフォルダ未添付の Cowork 会話（CWD が `/tmp`、`/var/folders`、または `/private/tmp` 配下）で実行された場合、永続的な保存先（デフォルト: `$HOME/Documents/req-workspaces/<slug>/`）への書き込みを提案し、その後のセッションで Cowork にそのフォルダを添付するよう案内します。

### セキュリティ

外部資料は要約されて `references.md` に保存されます。一般的なシークレットパターン（API キー、パスワード、トークン、Bearer）に一致するコンテンツは URL のみを記録します。取り込まれた要約は専用セクションに隔離され、指示ではなく背景データとして扱われます。

## コントリビュート

`CLAUDE.md` にプレースホルダ一覧、SKILL.md とテンプレの同期ルール、スモークテストチェックリストをまとめています。

## ライセンス

MIT
