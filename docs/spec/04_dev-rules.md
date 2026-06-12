# 04 開発ルール

## ブランチ戦略

GitFlowベース（fast-forward運用）

```
main        本番リリース済みコード
develop     開発統合ブランチ
feat/#xxx   機能開発（イシュー番号を付ける）
fix/#xxx    バグ修正
```

### 運用フロー
```
feat/#123 → develop（fast-forward merge）→ main（リリース時）
```

- `main` への直接pushは禁止
- PRはすべて `develop` ブランチへ向ける
- リリース時のみ `develop → main`

---

## コミット規約

### フォーマット
```
<prefix>: <概要> #<イシュー番号>
```

### プレフィックス一覧

| プレフィックス | 用途 |
|--------------|------|
| `feat` | 新機能追加 |
| `fix` | バグ修正 |
| `docs` | ドキュメント変更 |
| `style` | フォーマット・スタイル修正（動作に影響なし） |
| `refactor` | リファクタリング |
| `test` | テスト追加・修正 |
| `chore` | ビルド・設定・依存関係の変更 |

### 例
```
feat: マップのトグル切替機能を追加 #12
fix: オフライン時に避難所データが表示されない問題を修正 #34
docs: tech-stack.mdを更新 #5
```

### 強制ルール（commitlintで自動チェック）
- プレフィックス（feat/fix/docs等）は必須
- イシュー番号（#数字）は必須
- 上記を満たさないコミットはpre-commitフックで弾く

---

## 環境統一

### Node.jsバージョン
- `.node-version` ファイルでバージョンを固定
- nvm / mise / asdf いずれでも読み込み可能
- Windowsの場合はWSL環境を使用すること

```bash
# nvm の場合
nvm use

# mise の場合
mise install
```

### パッケージマネージャ
- pnpm を使用（Corepackで自動管理）
- `npm install` / `yarn` は使用禁止

```bash
# 初回セットアップ
corepack enable
pnpm install
```

---

## CI（GitHub Actions）

| ワークフロー | トリガー | 内容 |
|------------|---------|------|
| `lint.yml` | PR作成・push | ESLint + TypeScript型チェック |
| `test.yml` | PR作成・push | Jestによるユニットテスト |

---

## VSCode推奨設定

`.vscode/extensions.json` で以下の拡張機能を推奨：
- ESLint
- Prettier
- TypeScript（公式）
- Expo Tools
- i18n Ally（多言語JSONの補完）

---

## PR運用

- PRのベースブランチは `develop`
- レビュー承認1名以上でマージ可
- CI（Lint・Test）が通っていることがマージ条件
