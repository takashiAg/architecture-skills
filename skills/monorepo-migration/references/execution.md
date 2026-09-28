# 実行基盤とスクリプトの整理

## 共通入口と実装を分ける

Node系でpnpmを採用したworkspaceでは、開発者・CI・AIが使う公開コマンドを `pnpm run <task>` に揃える。pnpmはコマンドの入口、sh／Node／TSは処理の実装であり、併用そのものは問題ではない。問題は同じタスクを異なる手順・設定で実行していること。

単一packageはそのmanifest、monorepoのrootはpackageタスクへの委譲と全体調整を担う。pnpmから呼ぶshが同じpnpmタスクを再度呼ぶ循環を作らない。非Nodeプロジェクトにはpnpmを強制せず、既存の適切な実行入口を統一する。

## 配置例

```text
repo/
├── package.json                      # 公開コマンドとpackageへの委譲
├── scripts/check-environment.mjs      # repoの開発環境チェック
└── apps/api/
    ├── package.json                  # app固有の実行コマンド
    ├── scripts/                      # appのcodegen等。必要な場合のみ
    └── src/
        ├── domain/
        ├── application/
        ├── outbound/
        └── inbound/
            ├── cli/import-orders.ts  # 引数・結果変換、ユースケースを呼ぶ
            └── bootstrap/import-orders.ts  # 配線と起動
```

業務バッチはinbound、開発支援はscriptsに置く。schema migrationは既存ORMの規定場所を維持してよい。配置を4ディレクトリへ合わせるためにツールの検出を壊さない。

appのmanifest抜粋例。`dist/` の構造、build方法、対象package名は実環境に合わせる。この例はbuild済みJSを動かす場合であり、そのまま使える雛形ではない。

```json
{
  "name": "@repo/api",
  "private": true,
  "scripts": {
    "orders:import": "node dist/inbound/bootstrap/import-orders.js"
  }
}
```

rootのmanifest抜粋:

```json
{
  "private": true,
  "scripts": {
    "orders:import": "pnpm --filter @repo/api --fail-if-no-match run orders:import"
  }
}
```

引数は公開コマンドから渡す。`pnpm run` ではscript名より後ろの引数が実行先へ渡る。`--dry-run` は自作CLI側で実装・検証して初めて有効になる。

```sh
pnpm run orders:import --input ./orders.csv --dry-run
```

相対ファイルパスをpackageの作業ディレクトリ基準にするか、利用者の起動位置基準にするかを決めて文書化する。root→filter→packageの委譲で起動位置を取り違えない。必要なら絶対パス入力を受け付ける。

## 実行契約

公開するタスクごとに次を定義する。存在しない機能は約束しない。

- runtimeとpackage managerのversion、対応OS、作業ディレクトリ。
- 入力引数、環境変数名、出力、成功・失敗のexit code。
- buildが必要か、事前生成物、DB接続などの前提。
- timeout、SIGINT／SIGTERM時の終了と子processの扱い。
- 更新系なら対象環境、再試行・冪等性・再開位置、必要なら副作用のないdry-run。

shellはOSコマンドの短い接続に限定し、複雑な引数処理やJSON加工はNode等のparserへ移す。bashを使うならbashと明示し、`sh` からbash専用構文を実行しない。失敗の伝播と引数の引用を守り、ユーザー入力をshell文字列へ連結しない。Nodeの子processは可能なら引数配列で起動する。

`set -e` 等だけで全失敗を検知できるとしない。失敗を `|| true` で消す、pipeの途中失敗を見逃す、子processのexit codeを無条件0にする実装を調べる。常駐processのwrapperはsignalと終了処理も検証する。

## タスクの種類を分ける

| 種類 | 例 | 実行上の扱い |
| --- | --- | --- |
| 読み取り検証 | lint、typecheck、test | 失敗を非0で返す。checkが不足している場合は明記 |
| ソース変更 | format:write、lint:fix、codegen | 検証タスクと分け、差分を見えるようにする |
| 成果物生成 | build | 依存順、入力、成果物パスを明示 |
| 常駐 | dev、worker | 起動・終了・signalを扱う |
| 外部更新 | migrate、import、deploy | 対象と副作用を明示。cacheで実行を省略しない |

必須タスクに `--if-present` を付けて未定義を成功扱いしない。再帰実行も全packageに必須scriptがある保証にはならないため、対象一覧とmanifestを照合する。重要なfilter指定は対象0件を失敗にする。

必須の前処理は明示したtask依存か公開コマンドに含め、暗黙のpre/post hookだけに頼らない。package manager変更時はhookの実行方針も比較する。build成果物が必要なテストをbuildと無条件並列にしない。

## ローカル・CI・本番

CIは同じ公開タスクを呼び、別の長いshellに処理を複製しない。CI固有の認証・cache設定・実行環境はCI側に残せる。CLIはlockfileに固定したローカル依存を使い、未固定の一時取得を標準実行経路にしない。[pnpm run](https://pnpm.io/cli/run)、[pnpm exec](https://pnpm.io/cli/exec)

本番containerではpnpmを同梱する必要はなく、同じbuild成果物を直接 `node` 等で起動してよい。開発用TS runnerやrepoのnode_modulesがなければ動かないartifactを避ける。共通入口の規約と本番runtimeの最小構成を混同しない。

## 整理の進め方と受入条件

1. package scripts、sh、CI、Docker、README、cronを列挙し、同じ処理の重複・呼び出し循環を調べる。
2. 現行コマンド→責務→配置→新しい公開コマンド→利用者の対応表を作る。
3. まず既存処理へ委譲する共通入口を作り、業務ロジック抽出や言語変更は分ける。
4. ローカル、CI、運用の呼び出し元を更新する。既存利用者がいれば期限付きの互換wrapperを残す。
5. clean環境で成功・引数エラー・内部失敗・対象0件・必要な終了処理を確認する。更新系は試験環境で重複とdry-runの無副作用も検証する。
6. 新旧の結果を比較し、参照がなくなった旧入口を除く。

移動だけで引数、cwd、環境変数、exit codeを変えない。変更が必要なら公開契約の変更として記録する。rollbackでは呼び出し元とタスク定義を一緒に戻す。

確認日: 2026-09-28。オプションは導入versionの[filtering資料](https://pnpm.io/filtering)等でも確認する。本ガイドは他スキルを必要とせず単独で適用できる。
