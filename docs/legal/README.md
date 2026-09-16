# 法務文書（プライバシーポリシー）

App Store / Google Play の必須提出物であるプライバシーポリシーの原文を管理する。

## ファイル

| ファイル | 状態 |
| --- | --- |
| `privacy-policy.ja.md` | ドラフト |
| `privacy-policy.en.md` | ドラフト |
| `privacy-policy.ko.md` | 未着手（翻訳待ち） |
| `privacy-policy.zh.md` | 未着手（翻訳待ち） |

アプリの対応言語は ja / en / ko / zh の 4 言語。App Store Connect に登録する URL は 1 つで足りるが、
主対象が外国人観光客であるため en は実質必須。ko / zh は公開までに翻訳する。

## 公開先

App Store Connect と Google Play Console は、アプリ未インストールでも開ける公開 https URL を要求する。
アプリのバンドル内テキストやリポジトリ上の Markdown では要件を満たさない。

想定している公開先は組織の GitHub Pages。

```
https://kobe-in-your-pocket.github.io/privacy/       # en（既定）
https://kobe-in-your-pocket.github.io/privacy/ja/
https://kobe-in-your-pocket.github.io/privacy/ko/
https://kobe-in-your-pocket.github.io/privacy/zh/
```

Client アプリの `src/features/legal/domain/privacy-policy.ts` がこの URL 構成を前提にしている。
公開先を変更する場合は同ファイルの `PRIVACY_POLICY_BASE_URL` も合わせて変更すること。

## 版数の運用

版数は制定日・改定日と同じ `YYYY-MM-DD` 形式を使う。

Client は同意した版数を端末内に保存し、保存された版数が現在の版数と異なる場合に再同意を求める。
そのため **本文を改定して版数を上げたら、必ず Client の `PRIVACY_POLICY_VERSION` も同じ値に更新する**。
更新を忘れると、改定後のポリシーに対する同意が取得されないまま利用が継続される。

軽微な誤字修正など再同意が不要な変更では版数を据え置き、`最終改定日` と改定履歴のみ更新する。
