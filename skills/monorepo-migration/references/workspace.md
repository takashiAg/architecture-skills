# Workspaceと境界

## 最小構成

```text
repo/
├── apps/
│   ├── web/
│   └── api/
├── packages/
│   ├── contracts/     # 本当に共有する場合のみ
│   └── config/        # 同じ設定を使う利用者がいる場合のみ
├── package.json
└── pnpm-workspace.yaml  # pnpmを選んだ場合
```

JS/TSの例。Python、Go等があるなら各ecosystemのmanifestとlockを保ち、Nodeのツールを強制しない。同じrepoでも同じruntimeや同時releaseにする必要はない。

既存appをまずそのまま移す。共有は複数の具体的利用者と変更理由が確認できてから。`common` に全モデルを集めない。DDDを使う場合も、層ごとのpackage分割より業務境界を保つ。

## 依存と公開口

- `apps → packages`、package間は循環しない依存を基本とする。appsを他packageからimportしない。
- 各packageに一意な名前、必要な依存、公開口を持たせる。repo内だからと相対パスで他packageのsrcへ入らない。
- 型aliasはビルド・実行時のpackage解決や依存宣言の代わりにならない。
- UIとserverで共有する型にDB client、秘密設定、server専用SDKを混ぜない。
- 内部libraryはソース公開か事前buildかを決める。TSソースを公開するなら全consumerが変換できるか、配備runtimeで動くか確認する。
- `exports`、型定義、ESM/CJS、生成物パスを一致させ、app単体の成果物でも解決できるようにする。

workspaceをpackage managerが定義し、task runnerがbuild順序やcacheを扱う。Turborepoはworkspace上に追加するものであり、monorepoの必須要件ではない。[公式構成ガイド](https://github.com/vercel/turborepo/blob/main/apps/docs/content/docs/crafting-your-repository/structuring-a-repository.mdx)

## ツール選定

| 状況 | 最初に検討する構成 |
| --- | --- |
| 少数package、短いbuild | workspaceと既存scripts |
| build/testの重複や待ち時間が大きい | 計測した上でtask graph・cache |
| 既にNx／Turboを運用 | 対応versionの公式手順で維持・調整 |
| 複数言語・異なるdeploy | 言語ごとの再現可能なタスクと明示的な依存 |

導入効果を「流行」ではなく、実行時間、誤った差分検知、設定重複などの観測値で説明する。
