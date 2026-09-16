# 03 アーキテクチャ

> **ステータス：初版（思想確定）**
> 02_tech-stack.md と 07_bounded-contexts.md を前提に、本プロジェクトのアーキテクチャ思想を確定する。
> 具体的なコード規約・テスト戦略・命名規則は別ドキュメントで切り出す。

---

## 0. このドキュメントの位置づけ

「**なぜこの構造を選んだか**」を残すことが目的。
このドキュメントを読めば、次が分かる状態にする：

- アーキテクチャ選定の理由
- 守るべき依存ルール
- どこに何を置くかの判断基準
- 採用しなかった選択肢と棄却理由

実装の細則（ファイル名規約・テストの書き方など）は本ドキュメントの対象外。

---

## 1. 設計思想

### 1.1 採用するアーキテクチャ

**Modular Monolith × Feature-scoped Clean Architecture**

- **Modular Monolith**: 1つのRN（React Native）アプリ（1デプロイ単位）の内部を、境界づけられたコンテキスト単位の独立モジュールに分割する
- **Feature-scoped Clean Architecture**: 各モジュールの内部を `domain / application / infrastructure / ui` の4層構造に分け、依存方向を一方通行に縛る

この2つは **直交する2軸** として機能する：

```
              モジュール内の依存方向
              （Clean Architecture）
                       ▲
                       │  ui ─► application ─► domain ◄─ infrastructure
                       │
                       │
   ───────────────────┼───────────────────► モジュール間の境界
                       │  （Modular Monolith）
                       │
              Tourism / Evacuation / Manner ...
```

- Modular Monolith が **横軸（コンテキスト境界）**
- Clean Architecture が **縦軸（依存方向）**
- 互いに競合せず、補完する

### 1.2 なぜこの組み合わせか

| 要件・制約 | 採用理由 |
|---|---|
| 境界づけられたコンテキストが既に明確（07） | DDDの境界 = モジュール境界 にそのままマッピングできる |
| 5〜10名のチーム、React初学者中心 | feature ごとに担当を割れる。「機能名」でコードを探せる |
| バックエンドが別リポ・REST通信 | API契約とドメインモデルを分離する強制力が必要 → infrastructure 層が翻訳役を担う |
| オフライン要件（避難所データ） | 同じドメインを「API版」「SQLite版」の2実装で持つ必要 → Repository パターンで自然に解ける |
| ユビキタス言語が整理済み（06, 07） | ドメイン層に Course / Spot / Genre を型として表現する価値が大きい |
| 将来の機能拡張・分離可能性 | モジュール単位で切り出せる構造を残しておく |

### 1.3 採用しなかった選択肢と棄却理由

| 案 | 棄却理由 |
|---|---|
| **Feature-Sliced Design（FSD）** | 7層分割は学習コスト過大。8コンテキスト × 7層では構造が崩壊 |
| **厳密な Clean Architecture（全社単一）** | 表示中心のアプリには重い。boilerplate でドメインが埋もれる |
| **Atomic Design 単独** | UIの整理しか解決せず、コンテキスト境界が消える |
| **型別フラット構成**（components / hooks / api 直下） | 5〜10名スケールで必ず崩壊する典型アンチパターン |
| **Bulletproof React そのまま** | 境界の強制が弱い。我々の構成では自動ガードレールが欲しい |

---

## 2. アーキテクチャの2軸

### 2.1 横軸：Modular Monolith

**1つのRNアプリ（= 1デプロイ単位）の中で、コードを境界づけられたコンテキスト単位に分割する。**

#### 3つの原則（Kamil Grzybek の整理に基づく）

1. **High Cohesion**: モジュール内は密結合（同じ理由で変わるものは集める）
2. **Low Coupling**: モジュール間は疎結合（公開APIだけで会話する）
3. **Encapsulation**: モジュール内部の実装は隠す

#### 通信ルール

- モジュール同士は **公開API（`index.ts`）経由でのみ通信**
- 内部実装（`domain/` `application/` `infrastructure/`）は他モジュールから import 不可
- 複数モジュールを跨ぐ画面（マップなど）は `src/widgets/` で合成する
- 真に横断する基盤（UI primitives、設定値、多言語化、位置情報など）は `src/shared/` に置く

### 2.2 縦軸：Feature-scoped Clean Architecture

**各モジュールの内部を4層に分け、依存方向を一方通行にする。**

| 層 | 責務 | 依存していい先 |
|---|---|---|
| **domain** | ドメインモデル・不変条件・Repository interface。純粋TypeScript | 何にも依存しない（外部ライブラリも含む） |
| **application** | ユースケース（複数のドメイン要素を組み合わせた手続き） | domain のみ |
| **infrastructure** | API・DB・外部ライブラリのアダプタ。Repository の実装 | domain のみ |
| **ui** | コンポーネント・hooks・スクリーン。React/RN への依存はここに閉じる | application, domain, shared |

#### 鉄則

- **domain は React も Expo も Drizzle も知らない**
- **infrastructure は domain で定義された interface を実装する**（依存性逆転）
- **ui は infrastructure を直接呼ばない**。application 経由、または UI hook 経由で interface を通す
- **application は薄くてよい**。単純な転送だけなら省略可（後述「軽量モード」）

---

## 3. ディレクトリ構造の指針

```
app/                            # Expo Router（ルーティングのみ・薄いシェル）
src/
  features/                     # 境界づけられたコンテキスト = モジュール
    tourism/                    # コアドメイン
      domain/                   # 純粋TypeScript
      application/              # ユースケース
      infrastructure/
        api/                    # REST クライアント
        db/                     # 自モジュールのDrizzleスキーマ
      ui/
        components/
        hooks/
      store/                    # Zustand: UI状態のみ
      index.ts                  # 公開API（外部からはここ経由）
    evacuation/
    manner/
    user/
    content-submission/
    qr-onboarding/
  widgets/                      # 複数モジュールを合成する画面（マップ等）
    map/
  shared/                       # 真に横断するもの
    ui/                         # デザインシステム
    lib/                        # 外部ライブラリ薄ラッパ（maps, location, sqlite, i18n等）
    config/                     # env, constants
    utils/
    types/
```

### 判断基準

| 迷ったら | 置き場所 |
|---|---|
| 1つのコンテキストに閉じる | `features/{context}/` |
| 2つ以上のコンテキストを合成する | `widgets/` |
| ドメインに無関係な汎用処理 | `shared/lib/` または `shared/utils/` |
| アプリ全体の見た目 | `shared/ui/` |
| ルーティング | `app/` |

---

## 4. 適用基準（フル / 軽量モード）

すべての feature に4層を強制すると、小さなモジュールでオーバーヘッドが勝つ。
**コアドメインはフル4層、それ以外は軽量モードを許容する。**

| コンテキスト | 区分 | 構成 |
|---|---|---|
| Tourism | コアドメイン | **フル4層** |
| Evacuation | コアドメイン | **フル4層** |
| Manner | コアドメイン | **フル4層** |
| User | 支援サブドメイン | **軽量**（application 省略可：domain + infrastructure + ui） |
| ContentSubmission | 支援サブドメイン | 初期は**軽量**、機能拡張時にフル化検討 |
| QrOnboarding | 支援サブドメイン | **超軽量**（domain 任意、ui + 薄い infrastructure） |
| Localization | 汎用サブドメイン | `shared/lib/i18n/` として配置（feature 扱いしない） |
| GeoLocation | 汎用サブドメイン | `shared/lib/geo/` として配置（feature 扱いしない） |

### 昇格・降格の基準

- 軽量モードのfeatureに **ドメインルールが芽生えたら** `domain/` と `application/` を追加してフル化
- フル構成だが `application/` が薄っぺらいだけなら省略してよい（→ 軽量に降格）

---

## 5. モジュール間通信の原則

### 5.1 公開API経由のみ

- 他モジュールから触れるのは `features/{context}/index.ts` で export したものだけ
- `domain/` `application/` `infrastructure/` の中身を直接 import するのは禁止

### 5.2 公開してよいもの・隠すもの

| 公開してよい | 公開しない |
|---|---|
| ドメインの読み取り専用型（`Spot`, `Course` など） | Repository interface |
| 表示用コンポーネントの一部（`SpotCard` など） | DTO・APIスキーマ |
| 読み取り系 hook（`useSpotSearch` など） | Zustand store の setter |
| 設定エントリポイント（`configureXxxRepositories`） | infrastructure の実装 |

### 5.3 循環依存の回避

- A → B → A のような循環依存は禁止
- 共有が必要になったら **共通の上位概念を `shared/` に切り出す**、またはイベント駆動で疎結合化する

---

## 6. 境界の自動強制（ESLint + CI）

人間の注意力に頼らない。**機械的に違反を検出する仕組み**を初期から入れる。

### 6.1 採用ツール

- **`eslint-plugin-boundaries`**: ディレクトリ間の import 制約を定義
- **GitHub Actions**: PR時に lint を自動実行。違反があればマージ不可

### 6.2 強制するルール

1. `features/*` 同士の直接 import を禁止（公開API経由を強制）
2. `shared/*` から `features/*` への依存を禁止（一方通行）
3. `features/*/domain/` は他層に依存禁止
4. `features/*/application/` は `domain/` のみに依存可
5. `features/*/infrastructure/` は `domain/` のみに依存可
6. `features/*/ui/` は `application/` `domain/` `shared/` のみに依存可

### 6.3 段階的導入

| フェーズ | 設定 |
|---|---|
| プロトタイプ期（最初の2週間） | boundaries は `warn`（CIで赤くしない） |
| 本実装以降 | boundaries を `error` に昇格、ブランチ保護で必須化 |

---

## 7. デプロイ単位の整理

| 区分 | 数 | 配信先 |
|---|---|---|
| RNアプリ | 1（iOS / Android 合算で1プロダクト） | App Store / Google Play |
| バックエンドAPI | 1 | サーバー（別チーム・別リポ） |
| 内部モジュール | N | RNアプリのバンドルに同梱 |

Modular Monolith の前提は **「内部はN個、出荷は1個」**。
将来のマイクロサービス化（モジュールを別アプリに切り出す）は、必要になった時点で再検討する。

---

## 8. データ層の役割分担

技術選定（02）で確定済み。アーキテクチャ的にどこに住むかを再整理：

| 技術 | 用途 | 配置 |
|---|---|---|
| **Zustand** | UI状態（マップモード、選択中スポット、言語） | `features/{context}/store/` |
| **TanStack Query** | サーバーデータのキャッシュ・同期 | `features/{context}/ui/hooks/` から呼ぶ |
| **expo-sqlite + Drizzle** | オフライン保存（避難所など） | `features/{context}/infrastructure/db/`（各モジュールが自分のテーブルを所有） |
| **AsyncStorage** | 軽量な設定値（言語選択など） | `shared/lib/storage/` |

### オフライン戦略（確定済み）

```
オンライン時  → バックエンドAPIから取得 → expo-sqlite に保存
オフライン時  → デバイス内 SQLite を直接参照（ネットワーク不要）
同期タイミング → アプリ起動時に差分チェック・更新（詳細は実装時に設計）
```

Repository パターンにより、ドメイン層は「データがAPIから来たかDBから来たか」を知らなくてよい。

---

## 9. 参考にした思想・実例

| 出典 | 何を参考にしたか |
|---|---|
| Shopify "Deconstructing the Monolith" | Modular Monolith の運用思想、API境界の重要性 |
| Kamil Grzybek "Modular Monolith with DDD" | モジュール内 Clean Architecture の構造、依存性逆転の実装パターン |
| Robert C. Martin "Clean Architecture" | 依存方向の一方通行ルール、ドメイン中心設計 |
| Eric Evans "Domain-Driven Design" | 境界づけられたコンテキスト、ユビキタス言語 |
| Bulletproof React（Alan Alickovic） | feature-based のディレクトリ構造、`shared/` の切り出し方 |

---

## 10. 未確定・後続検討事項

本ドキュメントで思想は確定したが、以下は実装フェーズで詳細を詰める：

- [ ] 命名規約（クラス・関数・ファイル名の細則）
- [ ] テスト戦略（domain / application / infrastructure / ui それぞれ）
- [ ] エラーハンドリング戦略（ドメインエラー vs インフラエラー）
- [ ] ロギング・監視
- [ ] 環境変数管理（dev / staging / prod）
- [ ] ESLint 設定ファイルの確定と CI 連携
- [ ] 模範実装の作成（Tourism を最初のテンプレートとする）

---

## 関連ドキュメント

- 01_overview.md — プロダクト概要
- 02_tech-stack.md — 技術スタック
- 04_dev-rules.md — 開発ルール・ブランチ戦略
- 06_ubiquitous-language-interview.md — ユビキタス言語ヒアリング
- 07_bounded-contexts.md — 境界づけられたコンテキスト
