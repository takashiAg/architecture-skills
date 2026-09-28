# npmからpnpm等への統一

## 統一の目的と範囲

選んだJS workspaceのpackage manager、runtime、lockfile、CIのinstall方式を揃える。monorepo化のためだけに既存managerを必ず交換しない。npm→pnpmが依頼されていればその変更を進めるが、frameworkや依存のmajor更新を混ぜない。

実バージョンと互換性を確認し、利用するpnpmを固定する。runtimeとmanagerの用意方法は既存環境と公式手順に合わせ、Corepackが常に同梱されると仮定しない。

## 移行手順

1. 各repoでmanifestと旧lockを保全し、現行install/build/test/startの結果を記録する。
2. workspace一覧を `pnpm-workspace.yaml` に明示する。例は `apps/*` と `packages/*`。入れ子packageの意図しない検出を避ける。
3. `package-lock.json` 等が残った状態で `pnpm import` を使い、生成されたlockと解決versionを比較する。既存workspaceは事前にworkspace定義が必要。[pnpm import](https://pnpm.io/cli/import)
4. 各packageの依存を明示し、内部依存には対応している場合 `workspace:*` 等を使う。npmにも同じ記法が使えると仮定しない。
5. scripts、CI、Docker、deploy、開発docsのinstall／execコマンドを更新する。repoで必要なCLIはdevDependencyに固定し、未固定の一時ダウンロードでCIを動かさない。
6. 既存node_modulesに頼らないclean環境で `pnpm install --frozen-lockfile` と既存の必要なタスクを実行する。検証後、旧managerのlockを対象workspaceから除く。

`pnpm import` は配置まで同一にする保証ではない。複数repoの独立した旧lockをJSONとして結合しない。各sourceで変換・検証してから統合し、単一workspace lockを生成する際の再解決差分を調べる。差分が大きければパッケージ統合とmanager変更を段階に分ける。

## よくある問題

| 兆候 | 調べること | 対処 |
| --- | --- | --- |
| npmでは動くのにpnpmでimport失敗 | 未宣言の推移依存を使っていないか | 使用packageに直接依存を宣言 |
| peer dependency警告 | consumerとの互換性、framework重複 | 必要範囲だけ整合。全体overrideで隠さない |
| native moduleが動かない | install script実行方針、ABI、build環境 | 使用versionの公式方針で必要なbuildを許可 |
| devで動くが配備で失敗 | workspace symlink先、dist、production依存 | appの配備成果物に必要なものを含める |
| shellがnpmを呼ぶ | CI/Docker/package scriptsの残存 | 文字列一括置換せず意味を確認して更新 |
| registryで内部名を取得 | ローカル依存の宣言漏れ | pnpmのworkspace protocolでローカル解決を要求 |

pnpmのworkspace protocolは、該当packageがローカルにない場合の意図しないregistry解決を防ぐ。[Workspace](https://pnpm.io/workspaces)

hoistを全体的に緩めて未宣言依存を隠すことを初手にしない。必要な互換設定は理由と範囲を記録する。pnpmのscript承認、deploy、注入設定はversion差があるため、導入versionの資料で確認する。

## 完了と戻し方

同一入力からlockを変更せずinstallでき、各appのbuild・test・startと本番相当の成果物が動くことを確認する。manager固定、workspace設定、scripts、CI、lockは整合した単位で戻す。旧lockだけ復活させても旧環境に戻ったとは限らない。

Python等の別ecosystemや独立管理が必要な領域のlockは維持し、その理由を明記する。

実行コマンドも統一する場合は [execution.md](execution.md) を読む。公開入口、引数・cwd・exit code、shell実装、CIからの呼び出し、旧コマンドとの互換性を合わせて整理する。
