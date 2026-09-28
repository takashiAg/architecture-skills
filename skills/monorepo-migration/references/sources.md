# 一次資料

確認日: 2026-09-28。GitHubの移行制約とツール設定は実行時にも再確認する。

- [Git: subtree](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.adoc): squashと履歴接続の違い。
- [git-filter-repo](https://github.com/newren/git-filter-repo): 履歴の書換えとfresh cloneでの操作。
- [GitHub: Issue transfer](https://docs.github.com/en/issues/tracking-your-work-with-issues/administering-issues/transferring-an-issue-to-another-repository): 所有者、公開範囲、権限、メタデータの条件。
- [GitHub: Repository transfer](https://docs.github.com/en/repositories/creating-and-managing-repositories/transferring-a-repository): repo全体の所有者変更。repo統合とは別。
- [pnpm: Workspace](https://pnpm.io/workspaces): workspaceとローカル依存の宣言。
- [pnpm: import](https://pnpm.io/cli/import): 旧lockからの変換。
- [Turborepo: Repository structure](https://github.com/vercel/turborepo/blob/main/apps/docs/content/docs/crafting-your-repository/structuring-a-repository.mdx): workspaceを土台にする構成。
- [Nx: Import project](https://nx.dev/docs/kb/import-project): 既存projectの取り込みと追加調整。

apps/packagesの配置、段階的切替、台帳による照合はこのスキルの推奨。特定ツールの採用を必須にしない。
