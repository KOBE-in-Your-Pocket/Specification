# 09 開発環境仕様

> **ステータス：初版**
> ペルソナと対応OSに基づく emulator マトリクスを「仕様」として明文化する。
> 実際のセットアップ手順は Client リポの `docs/dev-environment.md` を参照。

---

## 1. ターゲットとペルソナ

| 項目 | 内容 |
|------|------|
| ターゲット | 外国人観光客（神戸市訪問者） |
| ペルソナA | ミャンマー人留学生 20-30代 個人旅行 |
| ペルソナB | ドイツ人留学生 20-30代 個人旅行 |

留学生はターゲット（観光客）の代表的な設計人物像として位置付ける。機能セットは観光客向けのまま。

---

## 2. 対応OSの下限・メイン・最新

| プラットフォーム | 下限保証 | メイン開発 | 最新追従 |
|----------------|---------|-----------|---------|
| iOS | iOS 16 | **iOS 18** | iOS 19 |
| Android | Android 11 (API 30) | **Android 14 (API 34)** | Android 16 (API 36) |

「下限保証」 = 動作確認の最低ラインで、これ未満は非対応。
「メイン」 = 開発時の標準ターゲットで、PR レビュー時の動作確認はここ。

---

## 3. 推奨 emulator / simulator デバイス

### iOS Simulator（Mac M2/M3/M4 のみ）

| デバイス | OS | 用途 |
|---------|-----|------|
| iPhone SE 3 | iOS 16 | 下限保証・小画面・片手操作 |
| **iPhone 15** | iOS 18 | ⭐ メイン（独留学生主力） |
| iPhone 17 Pro | iOS 19 | 最新追従 |

### Android Emulator

| デバイス | API | システムイメージ | 用途 |
|---------|-----|----------------|------|
| Pixel 4a | 30 (Android 11) | google_apis arm64-v8a, RAM 3GB | 低スペック・ミャンマー留学生想定 |
| **Pixel 8** | 34 (Android 14) | google_apis_playstore arm64-v8a | ⭐ メイン |
| Pixel 9 | 36 (Android 16) | google_apis_playstore arm64-v8a | 最新追従 |

> Apple Silicon Mac は **必ず arm64-v8a イメージを使う**。x86_64 は Rosetta 翻訳で5〜10倍遅い。
> Windows / Intel Mac は x86_64 を使う。

---

## 4. ツールチェーンのバージョン

| ツール | バージョン | 管理方法 |
|--------|-----------|---------|
| Node.js | 22.16.0 | `.node-version` + `.tool-versions` |
| pnpm | 11.7.0 | `packageManager` フィールド + Corepack |
| Java (Android) | Corretto 21 (or OpenJDK 17/21) | `.tool-versions` |
| Xcode | 16+ | App Store（手動） |
| Android Studio | Koala 2024.1+ | 公式サイト（手動） |
| Expo SDK | 56 | package.json |

---

## 5. サポート言語

| 言語コード | 言語 | 翻訳調達 | 検証 emulator |
|----------|------|---------|--------------|
| ja | 日本語 | チーム | 共通 |
| en | 英語 | チーム | 共通 |
| zh | 中国語（簡体） | 機械翻訳 + 中国人レビュー | iPhone 15 / Pixel 8（CJK IME） |
| ko | 韓国語 | 機械翻訳 + 韓国人レビュー | 同上 |
| my | ビルマ語 | **要翻訳者確保（最大リスク）** | **Pixel 4a API 30（Myanmar Unicode 描画）** |
| de | ドイツ語 | 機械翻訳 + ネイティブ校正 | **iPhone 15（長文オーバーフロー）** |

---

## 6. チーム環境の一貫性確保

| 仕組み | 実体ファイル | 効果 |
|--------|------------|------|
| バージョンピンニング | `.node-version`, `.tool-versions`, `packageManager` | 各人のツールバージョンを自動で揃える |
| 単一情報源ドキュメント | `docs/dev-environment.md` | セットアップ手順の正解を1か所に集約 |
| セットアップ自動化 | `scripts/setup-emulators.sh` | AVD 作成を1コマンド化 |
| 環境チェック | `scripts/doctor.sh` | 「自分の環境が正しいか」を即検証 |
| CI / Dev 一致 | `.github/workflows/*.yml` が `.node-version` 参照 | CI と Dev のバージョン乖離を防ぐ |
| アプリビルド統一 | EAS Build → Dev Client 配布 | ネイティブビルドを個人でやらず、配布版で統一 |

---

## 7. 検証マトリクス（PR レビュー時の最低確認）

| 機能カテゴリ | iOS | Android |
|------------|-----|---------|
| 基本動作 | iPhone 15 / iOS 18 | Pixel 8 / Android 14 |
| 多言語（zh/ko/my/de） | iPhone 15 + 各言語切替 | Pixel 8 + 各言語切替 |
| 低スペック動作 | iPhone SE 3 / iOS 16 | Pixel 4a / Android 11（RAM 3GB） |
| 最新OS追従 | iPhone 17 Pro / iOS 19 | Pixel 9 / Android 16 |
| Burmese 描画 | — | **Pixel 4a / API 30 必須** |
| ドイツ語折返し | **iPhone 15 必須** | Pixel 8 |
| 位置情報権限 | iOS 18（精度選択UI） | Android 14（権限ダイアログ更新） |
| オフライン | iPhone 15 + 機内モード | Pixel 8 + 機内モード |

---

## 8. 未確定・後続検討

- Apple Developer Program の契約（年99ドル、Sprint 5 までに）
- EAS Build の無料枠運用 or Hobby 有料枠
- Google Maps Platform の API キー管理方法
- Burmese 翻訳者の調達ルート
- 実機テスト用デバイスの調達（学校から借りるか個人持ち寄りか）
