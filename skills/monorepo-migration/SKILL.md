---
name: monorepo-migration
description: 単一アプリや複数リポジトリをmonorepoへ段階的に移行し、構成、Git履歴、Issue・PR、package manager統一、依存関係、実行コマンド・scripts、CI、デプロイを整理する。monorepo移行の設計・実装・レビューや、それに伴うnpmからpnpmへの統一で使う。DDDの導入は前提にしない。
---

# Monorepo Migration

コードを同じ場所へ置くだけでなく、履歴、依存、開発・配備の再現性を保って移行する。monorepo化、package manager変更、業務設計変更は別の判断として扱う。

## 調査

依頼が計画・レビュー・実装のどこまでか確認する。指示ファイル、Git status、manifest、lockfile、runtime、workspace、CI、Docker、deploy設定を読み、旧パス→新パスの対応を作る。秘密値を表示せず設定名だけ扱う。

特定するもの:

- 移行元／先、各refと既定branch、履歴・tag・releaseの保持要件。
- app、共有library、所有者、公開package、独立deployの単位。
- Issue／PRの件数と状態、ラベル、milestone、Projects、リンク、移行できないメタデータ。
- package managerとruntimeの実バージョン、install/build/test/startの現状。
- CIと配備の作業ディレクトリ、トリガー、キャッシュ、環境変数名、成果物。

通常のローカル調査・修正を進める。履歴を保持するか曖昧なら保持を基本とし、元repoを書き換えないコピーで作業する。依頼がローカル移行だけならIssue移動・通知・archive・push等を勝手に実行しない。明示された外部操作はその範囲で進め、既存の許可を聞き直さない。

## 基本方針

- `apps/*` を実行・配備単位、`packages/*` を再利用単位の候補とする。既存の妥当な命名や非JSの配置は保つ。
- 依存をmanifestに明示し、公開API経由で利用する。app同士のソースimportや循環を作らない。
- package managerは選んだworkspaceで統一する。異なる言語のlockfileまで削除しない。
- まず既存のコマンドで動くworkspaceを作る。Turborepo／Nxは必要性があるときだけ追加する。
- Git履歴とホスティング側のIssue／PR／release資産は別々に移行する。
- フォルダ移動にDDD化、framework交換、依存の一斉更新を混ぜない。

## 参照の使い分け

- workspace・package設計: [workspace.md](references/workspace.md)
- 複数repo・履歴・Issue／PR: [history-and-issues.md](references/history-and-issues.md)
- 実行基盤・scriptsの配置・shとpnpmの整理: [execution.md](references/execution.md)
- npm→pnpm等の統一: [package-manager.md](references/package-manager.md)
- 段階的実装・CI・配備・戻し方: [migration.md](references/migration.md)
- 一次資料: [sources.md](references/sources.md)

## 出力と完了

計画では現状、目標tree、パスとpackage名の対応、選んだ履歴方式、Issue方針、段階ごとの検証とロールバックを示す。未確認事項を成功扱いしない。

レビューはファイル／設定を根拠に実害と最小修正を示す。実装依頼なら変更と必要な検証まで行う。clean環境でのinstall/build/testとapp単位の起動・配備成果物を確認し、外部切り替えが残るなら明記する。
