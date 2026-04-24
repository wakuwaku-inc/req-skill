---
domain: writing
version: 1
last_updated: 2026-04-24
---

# 執筆ガイドライン

このプラグイン内の全 `.md` ファイルに適用: `skills/req/SKILL.md`, `skills/req/SKILL.ja.md`, `CLAUDE.md`, `README.md`, `README.ja.md`, `templates/requirements.md`, および `docs/team-dd/` 配下の spec / plan ファイル。

## 言語分離ルール

- **英語:** Claude Code が実行時にロードするプロンプト (`skills/req/SKILL.md`)、メンテナ向けドキュメント (`CLAUDE.md`)、英語版 README、英語の spec / plan。
- **日本語:** スキルが実行時に出力するユーザー向けテキスト、日本語 README、`SKILL.ja.md` のドキュメント。
- 翻訳ドキュメント (`*.ja.md` 等) は英語原本の **追加の兄弟ファイル** として作成する。原本を置き換えてはならない。

## Tone & Voice

| 想定読者 | ファイル | レジスター |
| --- | --- | --- |
| Claude Code (実行時) | `SKILL.md` | 指示的・命令形 (「Read template」「Ask: …」)。順序が重要なルールは番号付き。 |
| メンテナ | `CLAUDE.md` | 規範的・簡潔。同期ルールと smoke-test はチェックリスト形式。 |
| エンドユーザー | `README.md` / `README.ja.md` | 会話的・説明的。サンプルブロック + セクション毎に 1〜2 文。 |
| 設計者 / コントリビュータ | `docs/team-dd/` | トークンエコノミー: 表 > 散文、不要な接続表現排除、根拠を繰り返さない。 |

## 用語集

| 用語 | 使用 | 回避 |
| --- | --- | --- |
| requirements / 要件 | 要件定義を指す全ての場面 | "spec" (仕様書・設計ドキュメント用に予約) |
| create mode / update mode | `/req` の 2 モードを区別するとき | 散文中の "new mode" / "diff mode" (spec の rationale 内なら可) |
| placeholder | テンプレートの `{{foo}}` トークン | "variable", "slot" |
| Notion MCP | Notion 用 MCP サーバー | "Notion API", "Notion plugin" |
| requirements doc | `/req` が生成するファイル | "spec file", "output" |

日本語の対応:

| 日本語 | 用途 |
| --- | --- |
| 要件 / 要件書 | requirements / 要件定義ファイル |
| 作成モード / 更新モード | create mode / update mode |
| プレースホルダ | placeholder |
| 書き戻し | write-back (update mode 専用) |

## プレースホルダ契約

`{{foo}}` の正本は `templates/requirements.md`。プレースホルダ一覧に言及する全ファイル (`SKILL.md`, `SKILL.ja.md`, `CLAUDE.md`) は、追加も省略もない **完全に同一の一覧** を保つ必要がある。テンプレートを変更した場合は 3 ファイル全てを同時編集する。詳細は `CLAUDE.md` の "Sync rules" を参照。

## ファイルペアリング

以下のペアは同期を維持する。英語側を変更したら同一コミットで日本語側も変更する:

- `skills/req/SKILL.md` ↔ `skills/req/SKILL.ja.md`
- `README.md` ↔ `README.ja.md`

`SKILL.ja.md` は **ドキュメント専用**。Claude Code は `SKILL.md` のみ自動ロードする。`SKILL.ja.md` には `name:` frontmatter を付けないこと。

## ユーザー向けリテラル文字列

ユーザー向け日本語プロンプト (例:「何について要件を作りますか？」) は **リテラル文字列** として扱う。`SKILL.md` と `SKILL.ja.md` の両方で同一表記を維持し、`SKILL.md` 側で英訳してはならない。スキルが実行時に発話する固定トークンとして扱う。

## 構造規約

- 見出しは `## セクション`, `### サブセクション` (ATX 形式)。setext (`===`, `---`) 形式は使わない。
- 見出し行の絵文字は `templates/requirements.md` および同テンプレから派生するユーザー向け出力要約のみ。SKILL / README / CLAUDE / docs の見出しには絵文字を付けない。
- 3 行以上の列挙は表、短い列挙は箇条書き。
- コードブロックには言語ヒント (` ```markdown `, ` ```bash `, ` ```json `) を付け、シンタックスハイライトを有効にする。
- 行末空白禁止。EOF には改行 1 つのみ。

## やってはいけないこと

- `SKILL.md` に複数段落の根拠説明を書かない。根拠は `docs/team-dd/specs/` 配下の spec に置く。
- 将来機能用のプレースホルダ行を先行追加しない (YAGNI)。
- 実動作と乖離する "使用例" を書かない。
- README に自動生成のマーケティング文句 (例: "Boost your productivity!") を混入させない。
