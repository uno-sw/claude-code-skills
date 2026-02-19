---
name: add-skill
description: Create a new Claude Code skill. Guides through skill design, writes SKILL.md with proper frontmatter, and creates supporting files as needed.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Bash(mkdir *)
argument-hint: [skill description or role]
---

# Add Skill

新しいスキルを作成する。スキルの仕様については [reference.md](reference.md) を参照。

## 手順

### 1. 要件のヒアリング

引数 `$ARGUMENTS` にスキルの役割・説明が指定されていれば、そこから目的を把握して設計を始める。なければユーザーに確認する。

以下を明確にする:

- **スキルの目的**: 何をするスキルか（`$ARGUMENTS` から読み取る）
- **起動方法**: ユーザーが手動で `/name` と呼ぶか、Claude が自動判断するか
  - 副作用のある操作（コミット、デプロイ、送信など）は `disable-model-invocation: true` を推奨
  - 知識・参照系は自動判断でよい
- **実行コンテキスト**: インライン（デフォルト）か、サブエージェント（`context: fork`）か
  - 会話履歴が必要ならインライン
  - 独立したタスクならサブエージェント
- **必要なツール**: `allowed-tools` で制限が必要か
- **引数**: `$ARGUMENTS` を使うか、どんな引数を受け取るか
- **サポートファイル**: テンプレート、例示、スクリプトなど追加ファイルが必要か

### 2. ディレクトリ作成

スキルの保存場所を決める:
- **個人用**（全プロジェクト共通）: `~/.claude/skills/<skill-name>/`
- **プロジェクト用**（このプロジェクトのみ）: `.claude/skills/<skill-name>/`

```bash
mkdir -p ~/.claude/skills/<skill-name>
```

### 3. SKILL.md の作成

以下のテンプレートを参考に `SKILL.md` を作成する:

```markdown
---
name: <skill-name>
description: <何をするか。Claude が自動判断するときはここのキーワードで判断される>
# 以下はオプション（必要なものだけ含める）:
# disable-model-invocation: true   # 手動起動のみにする場合
# user-invocable: false            # Claude のみが呼び出す場合
# context: fork                    # サブエージェントで実行する場合
# agent: Explore                   # context: fork 時のエージェント種別
# allowed-tools: Read, Grep        # 許可するツールを制限する場合
# argument-hint: [引数の説明]       # オートコンプリートのヒント
---

スキルの本体となる指示をここに書く。

## 使い方

...
```

### 4. SKILL.md の内容設計

**参照系スキル**（知識・規約・パターン）:
- 短くシンプルに。Claude がそのまま従う規約を書く
- 500行以内を目安。詳細は別ファイルへ

**タスク系スキル**（手順・自動化）:
- ステップバイステップで書く
- `$ARGUMENTS` でユーザー入力を受け取る
- 副作用があるなら `disable-model-invocation: true`

**動的コンテキスト注入** が必要な場合:
```markdown
現在のブランチ: !`git branch --show-current`
```

### 5. サポートファイルの追加（必要な場合）

- テンプレートや例示は別ファイルに切り出して `SKILL.md` から参照する
- スクリプトは `scripts/` サブディレクトリに置く

### 6. 動作確認の案内

作成後、ユーザーに確認方法を伝える:

- **直接起動**: `/skill-name`（または引数あり: `/skill-name 引数`）
- **自動判断確認**: `どんなスキルが使えますか？` と聞いてリストに表示されるか確認
- `disable-model-invocation: true` のスキルはリストに表示されない（手動起動専用）

## フロントマター早見表

| フィールド | 説明 | デフォルト |
|---|---|---|
| `name` | スラッシュコマンド名 | ディレクトリ名 |
| `description` | 用途の説明（Claude の自動判断に使われる） | 最初の段落 |
| `disable-model-invocation` | `true` で手動起動のみ | `false` |
| `user-invocable` | `false` で Claude のみ呼び出し可 | `true` |
| `context` | `fork` でサブエージェント実行 | インライン |
| `agent` | `context: fork` 時のエージェント種別 | `general-purpose` |
| `allowed-tools` | 許可ツールの制限 | 制限なし |
| `argument-hint` | オートコンプリートのヒント | なし |
