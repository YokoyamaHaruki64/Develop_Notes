# Wiki Generator — GitHub Actions 共通化設計

> 状態：設計ラフ（未実装） / 2026-11-15

## 目的

`development-wiki-generator` のビルド・出力処理を GitHub Actions の Reusable Workflow として提供する。利用側は Workflow テンプレートをコピーし、起動条件・設定ファイル・保存先などを指定するだけでよい。Generator の Git Submodule 登録は不要にする。

Markdown と `wiki.json` を正本とし、生成 HTML は設定に従って GitHub Pages に公開するか、リポジトリ内にコミットする。

## 責務分離

| 層 | 責務 |
| --- | --- |
| 利用側 Workflow | `push` / `pull_request` / `workflow_dispatch` 等の起動条件と公開条件、呼び出すバージョン・引数・権限の指定 |
| Generator 側 Reusable Workflow | Runner・Checkout・依存ツール・HTML生成・検証・成果物の出力 |
| Generator 本体 | `wiki.json` に基づく Markdown・図・内部リンクの処理と HTML 生成 |
| GitHub | Artifact保管、Pagesホスティング、Environment・Deployment管理、GitへのPush処理 |

**Pages向けとRepository保存向けは別のReusable Workflowとテンプレートにする。** 1つのWorkflowを `mode` で分岐させない。

## 想定リポジトリ構造

```text
development-wiki-generator/
├── build.mjs
├── assets/
├── templates/
│   ├── pages.yml                # 利用側にコピーするテンプレート
│   └── repository.yml
└── .github/workflows/
    ├── reusable-pages.yml      # Pages公開用
    └── reusable-repository.yml # Gitへの生成物保存用

Develop_Notes/
├── .github/workflows/wiki.yml # どちらかのテンプレートをコピー
├── wiki.json
├── WikiHome.md
└── HestiaDesign/
```

必要なら後から共通ビルド処理をComposite Actionへ切り出す。最初から共通化の階層を増やすことは必須としない。

## 共通のビルド処理

1. 利用側リポジトリの、イベントに対応するコミットをCheckout。
2. GeneratorリポジトリをRunnerの独立したパスにCheckout。Submoduleは不要。
3. Node.jsなど、Generatorが要求するビルド環境を準備。
4. 指定された `config`（既定 `wiki.json`）からHTMLを生成。
5. リンク・画像・設定および生成結果を検証。
6. 公開または保存に必要なファイルだけを収集。生成物の相対リンクが成立することを確認。

Reusable Workflowの `uses: owner/repo/.github/workflows/file.yml@ref` はWorkflow定義を読み込むだけで、そのリポジトリのソースをRunnerに自動展開しない。Generator本体は別途取得する。

### Generatorバージョン

- 利用側は `@v1` のようなメジャーバージョンタグ、必要に応じて特定タグ／コミットSHAを指定する。
- `v1` はGitHubがSemVerを自動解釈するものではなく、Generator管理者が安定版リリース時に参照先を更新するタグ。
- **Reusable WorkflowとGenerator本体は同じリリースに固定する。** Reusable Workflow内の `github` Contextは呼び出し元を指すため、単に `github.workflow_ref` から共通Workflowのバージョンを得られるとは想定しない。
- 取得するGeneratorのRefをどう一致させるかは実装時の要決定事項（メジャー別Workflow・明示的なRef引数・バージョン固定等）。

## Pages公開用

### 利用側テンプレートのイメージ

```yaml
name: Wiki Pages

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  wiki:
    uses: YokoyamaHaruki64/development-wiki-generator/.github/workflows/reusable-pages.yml@v1
    with:
      config: wiki.json
      publish: ${{ github.event_name != 'pull_request' && github.ref == 'refs/heads/main' }}
```

※ `@v1` と `publish` は将来の公開APIを想定した例であり、現時点では実装・タグ作成済みを意味しない。

### 内部処理

1. Build JobでHTML生成・検証。
2. 公開用ディレクトリを構築し、`actions/upload-pages-artifact` でPages Artifactをアップロード。
3. Deploy JobはBuild成功後に起動し、公開条件を確認。
4. `environment: github-pages` でデプロイ制限・承認・履歴を管理。
5. `actions/deploy-pages` がPages APIへデプロイ要求を送り、結果を確認。

PRではビルド検証だけを行い、原則として公開しない。Pages設定のSourceは「GitHub Actions」を使う。`pages: write` と `id-token: write` はデプロイ用Jobに限定する。

## Repository保存用

### 利用側テンプレートのイメージ

```yaml
name: Wiki Repository

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  wiki:
    uses: YokoyamaHaruki64/development-wiki-generator/.github/workflows/reusable-repository.yml@v1
    with:
      config: wiki.json
      target_branch: main
```

※ このYAMLも公開予定のインターフェース例であり、まだ動作するテンプレートではない。

### 内部処理

1. ソースの対象コミットからWikiを生成し、検証。
2. 指定された `target_branch` の最新コミットを取得する。既定値は `main`。
3. 生成したファイルのみ、保存先ブランチの所定パスへ反映。保存先の別ファイルを上書き・削除しない。
4. 差分があるときだけGitHub Actions Bot名義でコミットし、`GITHUB_TOKEN` によってPushする。
5. 差分がない場合はコミットしない。Push競合・権限不足・ブランチ保護は失敗として扱い、強制Pushしない。

ビルドと書き込みを別Jobにする場合は `actions/upload-artifact` / `actions/download-artifact` で成果物を受け渡す。Pages用のArtifact形式とは区別する。同一Jobで生成からコミットまで完結させる構成も比較対象とする。

### 保存先ブランチの扱い

- `target_branch` は**生成元ブランチではなく、生成結果の保存先ブランチ**。
- `target_branch: main` を標準運用とし、`docs` など既存の別ブランチも指定可能。
- 保存先ブランチが存在しない場合は、暗黙に新規作成せずエラーとする。
- 同じブランチにソースと生成物を置く場合、古いWorkflow実行の結果が最新のコミットを上書きしないよう、書き込み直前のHEADと生成元を検証する。
- 別ブランチ保存の場合も、ビルド元コミットの追跡と競合検出を行う。
- `GITHUB_TOKEN`で行った通常のPushは新しいWorkflow実行を原則発火させないため、自動コミットによる再帰実行を避けられる。ただしこの挙動に依存することを明示し、別の認証方式へ変更する場合は再検討する。
- `target_branch` 側の保護ルールでBotのPushが拒否される場合は、保護ルールを尊重する。
- Repository方式では生成物をGit管理対象にする。対象の `.gitignore` 設定を変更するか、生成物だけを明示的にステージする。

### 出力ファイルとリンク

現在のGeneratorは単一HTMLを出力するが、収録外のMarkdown等へのローカルリンクが残る場合がある。出力先が別ブランチでもリンクが成立するように、必要な付随ファイルの範囲を仕様化する。

## 入出力インターフェース（案）

| 引数 | Pages | Repository | 説明 |
| --- | --- | --- | --- |
| `config` | ○ | ○ | Wiki設定ファイル。既定は `wiki.json` |
| `publish` | ○ | — | Pagesへ公開するか。PR等では `false` |
| `target_branch` | — | ○ | 生成物を保存する既存ブランチ。既定は `main` |

生成先のファイルパスは、まず `wiki.json` の `output.path` を正本とする。Workflow固有の出力パス引数を追加する場合は設定の優先順位を明記する。

## 実装時に守る条件

- 利用側にGenerator Submoduleを要求しない。
- Workflowのビルド環境とGenerator取得処理はGenerator側が管理する。
- 公開・保存に不要なリポジトリファイル（`.git`、秘密情報、CI設定等）を配布しない。
- `pull_request` 起点の未信頼コードにリポジトリ書き込みを許可しない。
- 保存先 `target_branch` と生成元コミットは別の概念として扱う。
- Workflow内の相対パス、Artifact名、Environment名を混同しない。
- リリースタグとGenerator本体のRefの整合性を保つ。

## 移行方針

1. Generator側で2種類のReusable Workflowとテンプレートを実装する。
2. Generatorのバージョンタグを作成し、参照先を固定する。
3. 検証用リポジトリ／ブランチでPages公開とRepository保存をそれぞれテストする。
4. `Develop_Notes` の現行Submodule運用から切り替える。
5. 使用しなくなったSubmodule・旧Workflowを除去する。

この資料は実装計画のラフであり、**既存Workflowを変更する指示ではない**。特に `Develop_Notes` の現行CIとPages公開設定は、切り替えを検証するまでは維持する。

## 参考

- [GitHub Docs: Reusing workflows](https://docs.github.com/en/actions/how-tos/sharing-automations/reusing-workflows)
- [GitHub Docs: Using custom workflows with GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages)
- [GitHub Docs: GITHUB_TOKEN behavior](https://docs.github.com/en/actions/concepts/security/github_token)
