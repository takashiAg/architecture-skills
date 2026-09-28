# Git履歴とIssue・PRの移行

## まず保持対象を分ける

コミット、branch、tag、署名、LFS、submoduleはGit側。Issue、PR review、Projects、release添付、保護設定、webhook、CI secretsはホスティング側。Gitのcloneやmergeで後者まで移ったとしない。

source URL、移行対象refとSHA、取り込みprefix、方式、保管先を台帳に記録する。元repoと移行先の未コミット変更を保護し、履歴加工は使い捨てcloneで行う。外部切り替えが許可されていない段階ではローカルで検証可能な成果物まで作る。

## 履歴方式の選択

| 方式 | 保持・変更されるもの | 適する場合 |
| --- | --- | --- |
| 同じrepo内で移動commit | 過去のcommit IDを維持。過去は旧パス | 単一repoの再配置 |
| squashなしのsubtree add | 選んだrefに到達する元commitを接続。過去treeは旧配置 | SHAを保ち複数repoを取り込む |
| filter-repoでprefixを書換えてmerge | 過去も新prefix。tree/parent変更でSHAが変わる | 過去パスの一貫性が重要 |
| snapshot／squash取り込み | 移行先では詳細な祖先履歴を保持しない | 履歴を元repoに残すと明示的に決めた場合 |

subtreeに `--squash` を付けて「全履歴を保持」と説明しない。取り込んだbranchの履歴と全branch／tagの保存を混同しない。[git-subtree](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.adoc)

元SHAを残す取り込みの例。source、ref、prefixを調査結果に置き換え、cleanな移行用cloneで実行する。これはローカルcommitを作る操作。

```sh
git subtree add --prefix=apps/legacy-api /path/to/source-repo main
```

過去のパスを書き換える場合は使い捨てのfresh cloneで `git filter-repo --to-subdirectory-filter apps/legacy-api` 等を検討する。対象ref、commit-map、署名とtagへの影響を確認し、元repoへforce pushしない。履歴加工後のcommit/tag署名がそのまま有効だと期待しない。[git-filter-repo](https://github.com/newren/git-filter-repo)

取り込み後は以下を確認する。

- SHA保持方式ではsource tipが移行先HEADの祖先になっているか。`git merge-base --is-ancestor SOURCE_SHA HEAD` 等で確認。
- 取り込み直後のprefixのtreeがsource tipのtreeと一致するか。配置変更後の修正は別commitにして比較可能にする。
- 代表的なファイルのlog／blameを追えるか。subtree接続やmergeをまたぐ履歴は単純な `--follow` だけでは見えない場合があるため、元SHAと旧パスからも確認する。
- tag衝突を解消する名前空間、未取り込みbranchの保存先、releaseとの対応。
- LFS実体、submoduleの参照先とアクセス権。通常のGit bundleだけでLFS実体まで保全したとしない。

## Issue移行

移行台帳は `source URL / source ID / state / destination URL / action / verification` を持つ。再実行時はsource IDで照合して二重作成しない。件数だけでなく、担当者、コメント、添付参照、ラベル、milestone、Projects項目を標本確認する。

GitHubのnative transferは、確認時点ではopen Issue、同一ユーザー／組織のrepo間、双方のwrite権限が条件。private→publicには使えない。コメントと担当者は保持される。ラベルは名前、milestoneは名前と期日の一致が必要。旧URLは転送され、mention先には通知される。[GitHub公式仕様](https://docs.github.com/en/issues/tracking-your-work-with-issues/administering-issues/transferring-an-issue-to-another-repository)

実行前に現在の制約を再確認し、移行先のラベル等を整える。許可済みの移行範囲で使うコマンド例:

```sh
gh issue transfer https://github.com/OWNER/SOURCE/issues/123 OWNER/DESTINATION
```

Issue番号の維持を前提にしない。元URLと新URLの対応を記録し、本文やdocsの裸の `#123` が移行先の別Issueを指さないか確認する。過去commitの本文はSHA保持のために書換えず、対応表から追跡する。

closed Issueは元repoを参照できる形で残す選択を基本とし、移すためだけに一括reopenしない。別owner等でnative transferが使えない場合は、旧repo維持＋索引か、明示的に選んだ再作成を使う。再作成は元の作者・時刻・review状態を完全保持できるとは限らず、出典を明示する。private情報を公開先へ複製しない。

## PR・運用資産

PRはIssue transferと同じ方法で移せると仮定しない。旧repoでmergeして最終差分を取り込む、またはbranchを移して新PRを作り旧PRを参照する。新PRではreview・承認・checksの履歴が同一にはならない。closed PRは元repoの監査記録として残せる。

release添付、Projects、自動化、branch protection、CODEOWNERS、deploy key、webhook、環境設定は別のチェック項目にする。Git tagだけではrelease資産は移らない。repo全体の所有者transferは複数repoを一つに統合する操作ではない。

## 切り替えと復旧

最終取り込みSHAと受付先を決め、進行中PR・未移行Issueを台帳に残す。旧repoのarchiveは移行検証と書込み先切替後に行う対象操作として扱う。履歴保持のために元repoを削除しない。

コードのrevertではIssue transfer、通知、外部設定は戻らない。戻す対象、戻せない通知・採番、旧repoの再開手順を分ける。外部操作の前に、対象台帳と実行結果を照合できる状態にする。
