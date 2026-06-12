# KOBE in Your Pocket - Specification

本プロジェクトの仕様・設計ドキュメントを管理するリポジトリです。

## ドキュメント構成

```
docs/spec/
  01_overview.md                   # プロダクト概要
  02_tech-stack.md                 # 技術スタック選定
  03_architecture.md               # アーキテクチャ（確定次第更新）
  04_dev-rules.md                  # 開発ルール・ブランチ戦略・コミット規約
  05_team.md                       # チーム構成
  06_ubiquitous-language-interview.md  # ユビキタス言語ヒアリングシート
```

## ブランチ戦略

| ブランチ | 用途 |
|---------|------|
| `main` | 承認済みドキュメントの最新版 |
| `develop` | レビュー中・統合中のドキュメント |
| `feat/#xxx` | 個別ドキュメントの追加・更新作業 |

## コミット規約

```
<prefix>: <概要> #<イシュー番号>

例）
docs: ユビキタス言語ヒアリングシートを追加 #3
feat: アーキテクチャ仕様を追加 #7
fix: tech-stackの誤記を修正 #10
```

## ルール

- `main` / `develop` への直接pushは禁止
- マージにはPRと1名以上の承認が必要
