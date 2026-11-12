# LEIR — 企画・仕様の索引

このディレクトリの `Design/*.md` がゲーム仕様の正本です。AI・開発者は必要なMarkdownだけ参照し、HTMLを仕様変更の正本として編集しないでください。

## 資料の一覧

| 用途 | 正本 |
| --- | --- |
| 概要・制作スコープ | [ゲーム概要](Design/Overview.md) |
| キャラクター・画像 | [キャラクター](Design/Characters.md) |
| コンボ・攻撃・ステップ | [戦闘](Design/Combat.md) |
| 大群の規模と体験 | [戦闘規模](Design/CombatScale.md) |
| 攻撃モーションの共通仕様 | [共通モーション](Design/AttackMotion_Common.md) |
| 弱 → 弱 → 強 | [弱弱強](Design/AttackMotion_WeakWeakStrong.md) |
| 弱 → 強 → 強 | [弱強強](Design/AttackMotion_WeakStrongStrong.md) |
| 単発強 | [単発強](Design/AttackMotion_StandaloneStrong.md) |
| 撮影用チェック | [撮影チェック](Design/AttackMotion_ShootingChecklist.md) |
| 敵・ボス | [敵仕様](Design/Enemies.md) |
| 部屋・通貨・ショップ | [ローグライト](Design/Roguelite.md) |
| ステージ制作共通 | [共通ステージ](Design/Stage_Common.md) |
| 草原 | [草原](Design/Stage_Grassland.md) |
| 遺跡 | [遺跡](Design/Stage_Ruins.md) |

## 人間向けWiki

閲覧用の [開発Wiki](../index.html) は、リポジトリ直下の `wiki.json` と `Design/*.md` から自動生成しています。旧 `Web/index.html`・`Web/style.css`・`Web/script.js`・`Web/build.js` の編集は不要です。

- **確定**: 現段階の採用方針
- **仮**: プロトタイプで変更する可能性のある方針や数値
- **未決定**: まだ決めていない事項
- **拡張候補**: 初期実装後に検討する事項

更新時は対象の `Design/*.md` を先に変更し、その後 `node development-wiki-generator/build.mjs --config wiki.json` でルートのHTMLを再生成してください。古いHTML固有のハッシュURLは新Wikiのリンクと互換ではありません。新しい参照にはMarkdownのパス、または新Wikiの文書リンクを使ってください。
