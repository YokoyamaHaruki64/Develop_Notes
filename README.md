# Develop Notes

HestiaEngineとLEIR（仮題）の設計資料・開発ノートです。

- **開発Wiki（GitHub Pages）**: https://yokoyamaharuki64.github.io/Develop_Notes/
- **Wiki閲覧用HTML**: [index.html](index.html)
- **HestiaEngineの設計資料**: [HestiaDesign/](HestiaDesign/)
- **LEIRの仕様書**: [MusouGame/Design/](MusouGame/Design/)
- **LEIRの編集用索引**: [MusouGame/README.md](MusouGame/README.md)

## 生成と更新

Markdownが仕様の正本です。閲覧用HTMLは `wiki.json` とMarkdownを元に [development-wiki-generator](https://github.com/YokoyamaHaruki64/development-wiki-generator) で生成します。旧 `MusouGame/Web/` のHTML手編集・バンドル方式は廃止しました。

```sh
git submodule update --init
node development-wiki-generator/build.mjs --config wiki.json
node development-wiki-generator/build.mjs --config wiki.json --check
```

Node.js 24での利用を推奨します。画像と内部リンクを含め、編集後は `--check` で生成物の一致を確認してください。
