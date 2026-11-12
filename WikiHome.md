# 開発Wiki

HestiaEngineとLEIR（仮題）の設計資料・開発ノートをまとめています。Markdownを正本とし、このページはその閲覧用に生成しています。

## HestiaEngine

C++で制作しているゲームエンジンの設計・実装方針をまとめています。

- [クラス設計の基本方針](HestiaDesign/ClassDesign.md)
- [Subsystem / Boundary API / Game Facade](HestiaDesign/SubsystemApiFacadeDesign.md)
- [TransformProcessの並列化設計](HestiaDesign/TransformProcessParallelization.md)
- [命名規則](HestiaDesign/NamingConvention.md)

## LEIR（仮題）

多数の敵と戦う3Dアクションゲームの企画・仕様をまとめています。実装やプロトタイプで変更する可能性がある箇所は、各文書の「確定」「仮」「未決定」などの表記を優先してください。

- [ゲーム概要](MusouGame/Design/Overview.md)
- [戦闘仕様](MusouGame/Design/Combat.md)
- [大群殲滅の規模とゲーム体験](MusouGame/Design/CombatScale.md)
- [キャラクターとビジュアル](MusouGame/Design/Characters.md)
- [敵・ボス仕様](MusouGame/Design/Enemies.md)
- [ステージ共通仕様](MusouGame/Design/Stage_Common.md)

## 閲覧・編集について

左のメニューから各文書を開けます。仕様の更新は、このHTMLではなく対応するMarkdownで行ってください。新しい開発Wikiは [development-wiki-generator](https://github.com/YokoyamaHaruki64/development-wiki-generator) を使用しています。
