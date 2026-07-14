---
name: notion-create
description: 嶋本さんのNotionワークスペースに新しいDB・ページ・仕組みを作るときのスキーム集。「Notionに〜を作りたい」「新しいDBがほしい」「〜を管理したい」という依頼で使う。命名規則・最小プロパティ原則・標準DDLテンプレート・リレーション先ID一覧を含む。
---

# Notion新規作成スキーム

## 作成の原則

1. **最小構成で始める。** 初期プロパティは「タイトル＋日付＋URL＋メモ＋リレーション1本」まで。セレクトは3〜5択、ステータスは3段階（未着手/進行中/完了 など）。プロパティは使いながら足す
2. **新DBを作る前に既存DBのビューで済まないか確認する。** DBが増えるほど管理が破綻する（過去にDBマスターというDB管理用DBが必要になった経験あり）
3. **置き場所は 🎓 大学院ページ（`3998b8de-3e4f-8144-9257-d7e32c4965ea`）配下。** メインページに直接DBを置かず、サブページを1枚挟む（例:「授業ノート」「面談ノート」ページ）。メインページには授業DBなど毎日見るものだけ
4. **スマホ前提。** 1カラム・リストビュー中心・列レイアウト禁止。デフォルトで日付降順のリストビューを付ける
5. **資料はGoogle Driveに置き、NotionはURL欄でリンクする**

## 命名規則

- DB名: 絵文字＋日本語（例: `📖 授業ノート`）。ページ名の**先頭にスペースを入れない**（検索性が落ちる）
- URL欄の名前は「URL」で統一（MCPからは `userDefined:URL` で読み書き）
- 数式による表示用列は「〜名」「〜状況」など役割がわかる名前にする

## リレーション先ID一覧（既存DBのdata source ID）

| DB | data source ID |
|---|---|
| 🎓 授業 | `af82580f-aa93-4190-aa38-b5530a110409` |
| 📖 授業ノート | `a333b03d-30c1-410c-951d-eb67bfb2bba6` |
| 🗣 面談ノート | `7ec97e24-f0ee-417d-8c63-8e8959596232` |
| 📚 論文・資料 | `d75fd4e8-e0ba-482e-a5e9-f81accd74597` |
| 👥 人脈 | `df621b0e-106f-403a-871a-30a0ea0a5e7d` |
| 🏛 学会 | `e011d247-28d6-4ed8-bca6-2df8d432132b` |
| 📄 参加記録 | `7c050b9a-dae1-4a58-9995-a9a00c9aa8bf` |

人と紐づくDBを作るときは 👥 人脈へ、授業と紐づくなら 🎓 授業へ、`RELATION('<ds_id>', DUAL '逆側の列名')` の双方向リレーションで繋ぐ。

## 標準DDLテンプレート（notion-create-database / update-data-source用）

### 汎用ノートDB（面談ノートと同型。議事録・読書ノート等に流用）
```sql
CREATE TABLE (
  "タイトル" TITLE,
  "日付" DATE,
  "相手" RELATION('df621b0e-106f-403a-871a-30a0ea0a5e7d', DUAL '<逆側列名>'),
  "URL" URL COMMENT 'Google Drive等の資料リンク',
  "メモ" RICH_TEXT
)
```

### 記録・ログ系DB（参加記録と同型）
```sql
CREATE TABLE (
  "タイトル" TITLE,
  "日付" DATE,
  "種別" SELECT('種別A':blue, '種別B':purple, '種別C':green),
  "URL" URL,
  "メモ" RICH_TEXT
)
```

### 文献・資料系DB（論文・資料と同型）
```sql
CREATE TABLE (
  "タイトル" TITLE,
  "著者" RICH_TEXT,
  "年" NUMBER,
  "カテゴリ" SELECT('理論':purple, '方法論':blue, '事例研究':green, 'レビュー':orange),
  "タグ" MULTI_SELECT(),
  "読了状態" SELECT('未読':red, '途中':orange, '読了':green),
  "URL" URL,
  "メモ" RICH_TEXT
)
```

学期系のセレクトを作る場合は阪大ターム制に合わせる:
`MULTI_SELECT('春':yellow, '夏':orange, '秋':brown, '冬':default, '春夏':blue, '秋冬':purple, '通年':gray, '集中':red)`

### 個別スキル用DBテンプレート（`skills-` リポジトリの各スキルで定義済み）

`skills-` リポジトリの一部スキルは、自分専用のNotion DBスキーマを
`references/notion_db_schema.md` として既に確定させている。これらは
「Notion MCP接続時にこのスキーマでDBを作って」とスキル側から一度だけ提案される
設計で、まだ実際には作成されていない（計画中）。notion-createから直接
依頼された場合も、定義がずれないよう以下のテンプレートをそのまま使う。
プロパティ定義の正本はあくまで `skills-` リポジトリ側なので、スキーマを
変更するときは両方を同時に直す。

#### 用語集DB（research-glossaryスキル用）
正本: `skills-` リポジトリ `Academic/research-glossary/references/notion_db_schema.md`
```sql
CREATE TABLE (
  "用語" TITLE,
  "分野タグ" SELECT('プライバシー・監視':red, '動物倫理・人と動物':orange, '研究方法(質的)':blue, '研究方法(量的・統計)':purple, '学術一般':green, 'その他':gray),
  "理解度" SELECT('説明できる':green, 'なんとなく':yellow, '要復習':red),
  "出典" RICH_TEXT,
  "追加日" DATE
)
```
ページ本文はDBの構造ではなくスキル側テンプレート（一言定義/詳しい説明/使用例/関連用語/自分の研究との関わり）をそのまま貼る運用。作成後は命名規則（絵文字＋日本語）に合わせて表示名を検討し、data source IDを上のリレーション先ID一覧に追記する。

#### タスクDB（task-triageスキル用）
正本: `skills-` リポジトリ `personal/task-triage/references/notion_db_schema.md`
```sql
CREATE TABLE (
  "タスク名" TITLE,
  "カテゴリ" SELECT('仕事':blue, '大学院':purple, '私用':green),
  "緊急度" SELECT('至急(今日中)':red, '今週':orange, 'それ以降・期限なし':gray),
  "所要時間" SELECT('短(〜15分)':green, '中(〜1時間)':yellow, '長(1時間超)':red),
  "期限" DATE,
  "状況" SELECT('未着手':gray, '進行中':blue, '完了':green, '待ち':orange),
  "待ち先" RICH_TEXT COMMENT '「待ち」のときのみ(例: ○○事務所の回答待ち)',
  "最初の一手" RICH_TEXT COMMENT '長タスクのみ(15〜30分でできる最初の一歩)',
  "出どころ" SELECT('思いつき':gray, 'ゼミ宿題':purple, '打合せ宿題':blue, 'その他':default),
  "追加日" DATE
)
```
完了タスクは削除・アーカイブせず、ビューのフィルタで隠すだけにする（スキル側の運用ルール）。作成後はdata source IDを上のリレーション先ID一覧に追記する。

## 「親アイテムごとの絞り込みビュー」を作る手順（重要）

ビューDSLはリレーションフィルターを無反応で破棄するため、以下の手順で作る:

1. 子DBに数式列を追加:
   `ADD COLUMN "親名" FORMULA('prop("親リレーション").map(current.name).join(", ")')`
2. 親アイテムのページに `notion-create-view`（`parent_page_id`＋`data_source_id`）でリンクドビューを作成
3. `configure` に `FILTER "親名" CONTAINS "<親のタイトル>"` を指定（タイトルが他と部分一致する場合は `=` で完全一致にする）
4. 注意: このビュー内の「+新規」ではリレーションが自動セットされない。追加は親ページのリレーションプロパティの＋から行う運用を案内する

## 作成後チェックリスト

- [ ] 日付降順のリストビューを付けたか
- [ ] URL欄（URL型）はあるか
- [ ] 親ページはメインページ直下ではなくサブページか
- [ ] 不要になった旧DB・旧ページを 🗄 アーカイブ（`3998b8de-3e4f-81f4-8c20-eb531d59a8a5`）へ移動したか
- [ ] ユーザーに「最初の1件」を入れてもらう案内をしたか（空のDBは使われなくなる）
