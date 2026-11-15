# Develop Notes

HestiaEngineとLEIR（仮題）の設計資料・開発ノートです。

- **開発Wiki（GitHub Pages）**: https://yokoyamaharuki64.github.io/Develop_Notes/
- **HestiaEngineの設計資料**: [HestiaDesign/](HestiaDesign/)
- **LEIRの仕様書**: [MusouGame/Design/](MusouGame/Design/)
- **LEIRの編集用索引**: [MusouGame/README.md](MusouGame/README.md)

## 生成と公開

Markdownと `wiki.json` が仕様の正本です。閲覧用HTMLは [development-wiki-generator](https://github.com/YokoyamaHaruki64/development-wiki-generator)（Git Submodule）を使って生成します。

GitHub Actionsは以下を自動実行します。

- **Pull Request**: Wikiを生成し、`--check` で検証する（公開しない）
- **mainへのPush／手動実行**: Wikiを生成・検証し、GitHub Pagesに公開する

`index.html` は生成物のためGitでは追跡しません。公開サイトには生成したHTMLとMarkdown・画像などの資料を含めます。旧 `MusouGame/Web/` のHTML手編集・バンドル方式は廃止しました。

### ローカルでの生成

Node.js 24を使用します。

```sh
git submodule update --init
node development-wiki-generator/build.mjs --config wiki.json
node development-wiki-generator/build.mjs --config wiki.json --check
```

生成後の `index.html` をブラウザで開けば、オフラインでもWikiを閲覧できます。Markdownや画像を編集した場合、公開用のHTMLはActionsが再生成します。

### GitHub Pagesの設定

リポジトリの **Settings → Pages → Build and deployment → Source** を **GitHub Actions** にしてください。追加のシークレットは不要です。
