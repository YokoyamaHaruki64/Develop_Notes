# LEIR 開発資料の記述・閲覧ルール

## 正本

- ゲーム仕様の正本は `Design/*.md`。確定・仮・未決定の状態、仕様ID、前提と例外を保持する。
- 画像素材は `Assets/`。本文に貼る画像はMarkdownの相対パスで参照する。
- [開発Wiki](../index.html) は生成結果であり、本文をHTMLに直接重複管理しない。

## 文書の書き方

- 文章・箇条書き・番号付き手順・表を優先する。見栄えのために仕様を分割しない。
- 図が必要な場合は元文書からSVG等を相対参照する。仕様IDや見出しを理由なく変えない。
- UIモックや仮の数値は確定仕様と混同しないよう明示する。

## ビルド

サイトのナビゲーションはリポジトリ直下の `wiki.json` にあり、HTMLは [development-wiki-generator](https://github.com/YokoyamaHaruki64/development-wiki-generator) により生成する。

```sh
git submodule update --init
node development-wiki-generator/build.mjs --config wiki.json
node development-wiki-generator/build.mjs --config wiki.json --check
```

旧 `MusouGame/Web/` にあったスタイル・UIモック・HTML本文の個別編集は廃止した。Markdownの内容を完全に表示することを優先し、旧HTML固有の対話UIや旧ハッシュの互換性は保証しない。
