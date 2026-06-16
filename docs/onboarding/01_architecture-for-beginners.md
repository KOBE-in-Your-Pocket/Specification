# 初学者向け：本プロジェクトのアーキテクチャ入門

> このドキュメントは、React Native / モバイル開発が初めての方が本プロジェクトに参加するときに最初に読むべき資料です。
> `docs/spec/` 配下の正式ドキュメントを読む前の "概念地図" として使ってください。

---

## 0. このドキュメントで分かること

- 「RN」「バンドル」「デプロイ」など基本用語の意味
- なぜこのアーキテクチャを採用したのか（思想）
- 自分が書いたコードがどこに置かれるべきかの判断基準
- 守らなければいけない最低限のルール

正式な設計仕様は `docs/spec/03_architecture.md` を見てください。本資料はその "やさしい入り口" です。

---

## 1. 用語集（最低限これだけ）

### RN（React Native）

**React Native の略**。Meta（旧Facebook）が作った、iOSとAndroidのアプリをJavaScript/TypeScriptで作れるフレームワーク。

- 1つのコードベースから iOS と Android の両方が動く
- React の書き方（コンポーネント・hooks）がそのまま使える
- 画面に出るのは「本物のネイティブUI部品」（ブラウザではない）

### Expo

RNの上に乗っかった「めんどくさいネイティブ設定を肩代わりしてくれるツールキット」。Xcode や Android Studio の知識が無くてもアプリが作れる。

本プロジェクトでは Expo を採用しています。

### バンドル

あなたの書いたソースコード（何百ファイル）を、機械が読める形に **1個にまとめた成果物**。Metro という変換器が作ります。

```
src/spot.ts、src/SpotCard.tsx、その他大量のファイル
        ↓ Metro
    index.bundle  ← これがバンドル（1個のJSファイル）
```

このバンドルが iOS / Android のアプリパッケージに埋め込まれて配布されます。

### デプロイ

開発したものを **ユーザーに届けること**。本プロジェクトでは App Store / Google Play への配信を指します。

### デプロイ単位

**1つのパイプラインで「ビルド → テスト → 配信」される塊**。今回は「RNアプリ1個」がデプロイ単位です。内部に Tourism / Evacuation / Manner などのモジュールが詰まっていますが、出荷時には1個にまとまって出ます。

### イシュー

GitHub Issue。「マップにピンを表示する」など、**1つの作業の単位**。複数のイシューをまとめて1回のデプロイで出すのが普通です。

```
イシュー：N個（feat/#42, feat/#43, ...） → デプロイ：1回（develop→main）
```

### ドメイン

「業務の中で扱う概念や規則」のこと。本プロジェクトの場合：

- 観光名所（Spot）には1つのジャンルと複数の属性がある
- コース（Course）は2つ以上のスポットからなる
- 避難所（EvacuationShelter）には開閉状態がある

こういう **「アプリが扱う世界そのもののルール」** をドメインと呼びます。

### Modular Monolith（モジュラーモノリス）

**1つの出荷物の中に、独立したモジュールが複数入っている構造**。本プロジェクトのアーキテクチャ。詳しくは §3。

### Clean Architecture（クリーンアーキテクチャ）

**コードを4層に分けて、依存方向を一方通行にする**設計手法。本プロジェクトはモジュールごとに採用。詳しくは §4。

---

## 2. このアプリの全体像

```
┌─────────────────────────────────────────────────────────┐
│ ユーザー（外国人観光客）のスマホ                          │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │ KOBE in Your Pocket（RNアプリ・1デプロイ単位）    │   │
│  │  ┌──────────┬──────────┬────────┬──────┐         │   │
│  │  │ Tourism  │Evacuation│ Manner │ ...  │ ←モジュール│   │
│  │  └──────────┴──────────┴────────┴──────┘         │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                          ↑ REST API
┌─────────────────────────────────────────────────────────┐
│ バックエンド（別リポジトリ・別チーム開発）                 │
└─────────────────────────────────────────────────────────┘
```

- **RNアプリ**: 私たちが作るもの。今あなたが居るリポジトリ
- **モジュール**: アプリの中の機能ごとの「区画」
- **バックエンド**: スポット情報などを提供するサーバー。**別チームが作る**

---

## 3. Modular Monolith とは（横軸）

### 一言で

**「1個のアプリを、複数の独立したモジュールで作る」** こと。

### 比喩：お弁当箱

```
┌─────────────────────────────────┐
│  お弁当（= 1個のアプリ）         │
│  ┌─────┬─────┬─────┬─────┐      │
│  │観光  │避難  │マナー│その他│      │ ← 仕切りがある
│  │おかず│おかず│おかず│おかず│      │
│  └─────┴─────┴─────┴─────┘      │
└─────────────────────────────────┘

→ 仕切りがあるから、観光のおかずに避難の汁が漏れない
→ でも、お弁当としては1個でユーザーに渡される
```

### なぜこの形か

| やったこと | 得たもの |
|---|---|
| モジュールに分けた | 担当を分けやすい、修正の影響範囲が広がらない |
| でも1個のアプリにまとめた | ユーザーは1個インストールするだけ、開発体制もシンプル |

### 必ず守ること

**モジュール同士は `index.ts` 経由でしか話してはいけない**。他モジュールの内部ファイルを直接 import するのは禁止。

```ts
// ❌ 悪い例
import { internalThing } from '../tourism/domain/spot';

// ✅ 良い例
import { Spot } from '@/features/tourism';   // index.ts 経由
```

---

## 4. Clean Architecture とは（縦軸）

### 一言で

**「1つのモジュールの中を4層に分け、依存方向を一方通行にする」** こと。

### 4つの層

```
ui ──► application ──► domain ◄── infrastructure
                          ▲
                          │ domain は何にも依存しない
```

| 層 | 何を書く | レストランで言うと |
|---|---|---|
| **domain** | ドメインのルール・型・interface（純粋TypeScript） | メニュー・レシピ |
| **application** | ユースケース（「検索する」「コースを作る」など） | 注文の手順書 |
| **infrastructure** | API通信、DB保存、外部ライブラリ呼び出し | 食材を仕入れる業者 |
| **ui** | 画面・コンポーネント・hooks | 接客スタッフ |

### 依存方向の鉄則

```
ui    →   application   →   domain      ←   infrastructure
          ↑ 下の層を使ってよい            ↑ domain だけ使ってよい
```

- **domain は何にも依存しない**。React も Expo も Drizzle も使わない。**純粋なTypeScript型と関数だけ**
- **infrastructure は domain の interface を実装する**。外の世界（API/DB）を domain の言葉に翻訳する
- **ui は infrastructure を直接呼ばない**。必ず application 経由か、UI hook で interface 越しに使う

### なぜこの形か

- **domain を純粋に保つ** → テストが書きやすい、別の技術に乗り換えやすい
- **infrastructure を交換可能にする** → 「APIから取る」を「SQLiteから取る」に差し替えられる（オフライン対応）
- **層を分ける** → どこを直せばいいか迷わない

---

## 5. 2軸の組み合わせ（このプロジェクトの設計）

```
              依存方向の縦軸（Clean Architecture）
                       ▲
                       │  ui ─► application ─► domain ◄─ infrastructure
                       │
                       │
   ───────────────────┼───────────────────► モジュール境界の横軸
                       │  （Modular Monolith）
                       │
              Tourism / Evacuation / Manner ...
```

- **横軸 = Modular Monolith**: 機能ごとにモジュールを分ける
- **縦軸 = Clean Architecture**: 各モジュールの中を層に分ける
- 互いに **直交する2つの軸** で、組み合わせると：
  - 機能を探すときは横軸を見る → Tourism なら `features/tourism/` を開く
  - 機能の中で何かを直すときは縦軸を見る → 「APIの呼び方を変える」なら `features/tourism/infrastructure/api/`

---

## 6. ディレクトリ構造（全体像）

```
app/                            ← Expo Router（画面のルーティングのみ）
src/
  features/                     ← モジュール（横軸）
    tourism/
      domain/                   ┐
      application/              │
      infrastructure/           │ ← 各モジュールの中の縦軸
      ui/                       │   （Clean Architecture）
      store/                    │
      index.ts                  ┘ ← モジュールの公開窓口
    evacuation/
    manner/
    ...
  widgets/                      ← 複数モジュールを合成する画面（マップなど）
  shared/                       ← 全モジュール共通の基盤
    ui/                         ← ボタンなど共通コンポーネント
    lib/                        ← maps, location, i18n などのラッパ
```

---

## 7. 具体例：「スポットを検索する」機能の流れ

ユーザーが検索ボックスに入力 → スポット一覧が表示される、を追ってみます。

```
1. [ui]              SearchScreen.tsx でテキスト入力を受ける
2. [ui hooks]        useSpotSearch(query) を呼ぶ
3. [application]     searchSpots(repo, query) ユースケースを呼ぶ
4. [domain]          domain で定義された SpotRepository.search(query) interface
5. [infrastructure]  SpotApiRepository.search() が REST API を叩く
                     結果のJSON → Spot 型（ドメイン）に変換
6. [application]     結果を返す
7. [ui hooks]        TanStack Query にキャッシュさせて返す
8. [ui]              画面に SpotCard を並べて表示
```

ポイント：
- `ui` は `application` を呼ぶだけ。**APIのことは知らない**
- `application` は `domain` の interface を使うだけ。**API か DB かは知らない**
- `infrastructure` だけが「実際に REST を叩く」を知っている

これが Clean Architecture の **依存性逆転**。

---

## 8. やってはいけないこと（落とし穴集）

| ❌ NG | なぜダメか |
|---|---|
| `features/tourism/` から `features/evacuation/` の内部ファイルを import | モジュール境界違反。`index.ts` 経由で |
| `domain/` で `import { View } from 'react-native'` | domain は純粋であるべき。UI層に移す |
| `domain/spot.ts` で `fetch()` を呼ぶ | I/O は infrastructure の責任 |
| `ui` から `infrastructure/api/spot-api-repository.ts` を直接 import | application または hook を経由する |
| スポットの名前を単なる `string` で持つ | LocalizedText を使う（多言語対応） |

これらは **ESLint で自動検出される**ようになっています。CIで赤くなるので、慌てず修正しましょう。

---

## 9. git の流れ（おさらい）

```
feat/#42 ──┐
feat/#43 ──┼─→ develop ──→ main ──→ App Storeリリース
feat/#44 ──┘   ↑              ↑           ↑
                統合           デプロイ準備    デプロイ実行
                （≠デプロイ）  （= 出荷確定）
```

- 1つのイシュー = 1つの `feat/#xxx` ブランチ
- 複数のfeat → `develop` で統合
- `develop → main` のタイミングが**1回のデプロイ**
- イシューを片付けてもすぐ配信されるわけではない（リリースは束ねて行う）

---

## 10. 困ったときの参考

- **正式仕様**: `docs/spec/03_architecture.md`
- **境界づけられたコンテキスト定義**: `docs/spec/07_bounded-contexts.md`
- **技術スタック詳細**: `docs/spec/02_tech-stack.md`
- **開発ルール**: `docs/spec/04_dev-rules.md`

不明点はチームの先輩に聞いてください。
「これってどの層に書くんですか？」は **良い質問**です。
