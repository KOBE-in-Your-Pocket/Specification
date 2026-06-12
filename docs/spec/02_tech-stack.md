# 02 技術スタック

## 採用技術一覧

| カテゴリ | 採用技術 | 選定理由 |
|----------|----------|----------|
| フレームワーク | Expo SDK（最新） + Expo Router | ファイルベースルーティング、Expoエコシステムで完結 |
| 言語 | TypeScript（strict） | 型安全、初学者のミス防止、チーム開発に必須 |
| パッケージマネージャ | pnpm + Corepack | バージョン統一、高速、ロックファイルで環境を揃える |
| ルーティング | Expo Router | Expo公式、Web経験者がNext.jsライクに扱える |
| UIステート管理 | Zustand | 学習コスト低、useStateに近い感覚で書ける |
| サーバーステート | TanStack Query（React Query） | APIデータのキャッシュ・同期を分離管理 |
| 多言語対応 | expo-localization + i18next + react-i18next | デバイス言語取得 + JSON翻訳管理、将来の言語追加が容易 |
| 地図 | react-native-maps | React Native事実上の標準、情報量が最多 |
| 位置情報 | expo-location | Expo管理ワークフロー内で動作、ネイティブ設定不要 |
| オフラインDB | expo-sqlite + Drizzle ORM | 避難所データの構造化保存・位置検索クエリ対応 |
| 軽量設定値保存 | AsyncStorage | 言語設定などキーバリューで済むもの |
| Lint | ESLint | コード品質統一 |
| フォーマット | Prettier | フォーマット統一 |
| pre-commit | Husky + lint-staged | コミット前の自動チェック |
| コミット規約 | commitlint | プレフィックス + イシュー番号の強制 |
| CI/CD | GitHub Actions | Lint・テスト・EASビルドの自動化 |
| ビルド・配信 | EAS Build | App Store公開のためのExpo公式クラウドビルド |
| テスト | Jest + @testing-library/react-native | ユニット・コンポーネントテスト |

---

## 技術選定の補足

### 状態管理の役割分担
```
Zustand        → UIの状態（マップモード切替、選択中スポット、言語）
TanStack Query → サーバーデータ（観光スポット一覧、避難所情報）
AsyncStorage   → 永続化が必要な設定値（選択言語など）
expo-sqlite    → 避難所データのオフライン保存（構造化・位置検索あり）
```

### オフライン戦略
```
オンライン時  → バックエンドAPIから避難所データ取得 → expo-sqliteに保存
オフライン時  → デバイス内SQLiteを直接参照（ネットワーク不要）
同期タイミング → アプリ起動時に差分チェック・更新
```

### バックエンド
- 別リポジトリ・別開発環境で並行開発
- 通信方式：REST API
- このリポジトリはフロントエンド（Expo）専用

---

## 未確定・後続検討
- UIライブラリ（NativeBase / Tamagui / Gluestack / 素StyleSheet）：要件確定後に選定
- プッシュ通知（地震速報等）：機能要件確定後に選定
- ユーザー認証：機能要件確定後に選定（Firebase / Auth0 / 自前APIなど）
