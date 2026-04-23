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

## コントリビュート

`CLAUDE.md` にプレースホルダ一覧、SKILL.md とテンプレの同期ルール、スモークテストチェックリストをまとめています。

## ライセンス

MIT
