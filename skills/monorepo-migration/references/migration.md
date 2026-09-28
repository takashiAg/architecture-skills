# 段階的移行・CI・デプロイ

## 段階と検証

| 段階 | 成果物 | 合格条件 | 戻し方 |
| --- | --- | --- | --- |
| 調査 | パス・ref・Issue・deploy台帳 | 現行タスクと所有者を把握 | 変更なし |
| 履歴取り込み | 移行branch、履歴方式の記録 | tree一致と必要な祖先／tag保持 | 移行branchを採用しない。元repo維持 |
| 配置・workspace | manifest、パス、依存 | 各appの既存build/test | 移動と設定を一単位で戻す |
| manager統一 | lock、runtime、CI install | clean環境で再現 | 旧lockと関連設定をまとめて戻す |
| CI・配備 | タスク、成果物、各app設定 | 下記の受入条件 | 検証済み旧artifactへ |
| 外部受付切替 | Issue対応表、旧repo案内 | 参照・所有者・受付先を確認 | 外部操作を個別に戻す |
| 定着 | 不要設定の削除 | 旧経路に依存がない | 保存した手順とartifactで復旧 |

移動だけの差分とロジック変更を分け、移行sourceの最終SHAを記録する。未コミット変更は勝手に捨てない。最初のappを通してから残りへ広げる。

## CI

- working-directory、lockパス、install、script名、成果物パスを更新する。
- app変更だけでなく、依存package、共有config、lockの変更がconsumerの検証を起動するか確認する。
- 変更のbase/headとcheckout履歴が不足すると差分検知が誤る。判定不能時に何も実行しない設定にしない。
- task graphに依存packageのbuild順を反映する。型チェック成功だけでruntime解決成功としない。
- cache keyの入力に結果を変える設定・runtime・環境変数を含め、成果物を定義する。
- publish、deploy、DB migration等の副作用を成功cacheで省略するタスクにしない。
- 最初は必要な全体検証を通し、affected-only最適化は依存検知を確認してから使う。

## デプロイ

appごとのbuild context、Docker COPY、workspace package、生成client、起動コマンド、環境変数名を確認する。repo全体を本番へコピーしないと動かない構成に安易にしない。

独立deployとreleaseを維持し、monorepoだから全appを同時releaseしない。公開libraryはversionと依存更新の方針を別に決める。DB migrationの実行責任を一つにし、各appの同時起動で重複実行させない。

本番外部設定の変更が依頼に含まれていなければ、ローカル／CIで検証できる設定と具体的な切り替え手順まで作る。

## 受入条件

1. 新しいclone相当の環境でinstallでき、lockが意図せず変わらない。
2. 既存のlint/typecheck/test/buildが必要範囲で通る。存在しないタスクを成功扱いしない。
3. 各appが単独起動し、本番相当artifactからworkspace依存を解決できる。
4. 共有package変更がconsumerのCIへ伝わる。判定できなければ全体検証へfallbackする。
5. Gitの必要な履歴とIssue対応を照合できる。
6. 旧artifactへの復旧条件、DB互換性、外部受付先の戻し方がわかる。

成功したコマンド、環境、未実施の配備確認を報告する。構成treeだけで移行完了としない。
