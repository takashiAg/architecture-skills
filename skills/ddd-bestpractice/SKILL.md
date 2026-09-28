---
name: ddd-bestpractice
description: 最小限のDDD構成を設計し、既存ディレクトリ・依存方向・業務ルール・アンチパターンをレビューして段階的な移行を支援する。DDDの設計、構成評価、DDDへのリファクタリングで使う。monorepoを基本とし単一アプリにも対応する。monorepo化だけの依頼や通常の機能実装には自動適用しない。
---

# DDD Best Practice

業務の言葉と不変条件を、小さなモデルと明示的な境界で守る。DDDはディレクトリの名称ではない。この4ディレクトリはDDDにポート／アダプターを組み合わせるための本スキルの規約であり、DDD一般の必須条件ではない。

## 調査と判断

対象の指示ファイル、manifest、workspace、入口、テストを読む。`rg --files` で対象を絞り、代表的な更新処理を入口から保存まで追う。必要なら参照系も追う。

- 新規設計、レビュー、移行計画、実装のどれを依頼されているか判断する。レビュー依頼だけでコードを変更しない。
- 重要な業務ルール、用語、変更で困る箇所、データ所有者を特定する。不明な業務仕様を確定しない。
- monorepoではリポジトリ、package、bounded context、デプロイの境界を別々に把握する。
- 単純なCRUDに集約やイベント基盤を足さない。責務と依存が既に整理されていれば、名前合わせだけの移動を勧めない。

## 最小構成と依存方向

新規の業務アプリ／packageの `src/` は `domain`、`application`、`outbound`、`inbound` を基本とする。空フォルダや使わない抽象化は不要。

| 場所 | 責務 | 内部で参照してよい相手 |
| --- | --- | --- |
| domain | 業務概念、値、状態遷移、不変条件 | 同じdomain内 |
| application | ユースケース、処理順序、外部機能のポート | domain、application内 |
| outbound | DB・外部API等のアダプターと変換 | applicationのポート、domain、outbound内 |
| inbound | HTTP・CLI・ジョブ等の入口と変換 | applicationの公開API、inbound内 |
| inbound/bootstrap | 起動、設定読込、実装の組み立て | application、outbound、inbound |

矢印はソースコード上の依存（import）。

```text
inbound ──→ application ──→ domain
outbound ─→ application
outbound ─────────────────→ domain
inbound/bootstrap ─→ inbound / application / outbound
```

実行時にapplicationが注入されたポートを呼びoutboundが動くことと、applicationがoutboundをimportすることを区別する。type-only import、再export、DIによる隠れた結合も確認する。

ポートは利用側が所有する。通常は `application/ports`。業務概念として必要なrepository契約はdomainでもよいが、domainからapplicationを参照しない。内側にHTTP、ORM、SDK、環境変数を漏らさない。標準ライブラリ等の技術非依存な道具は使える。

bootstrapの例外は配線だけ。HTTPハンドラーが直接DBを呼ぶ抜け道にしない。フレームワーク必須の入口配置は保ち、薄く委譲させる。

## 必要な参照だけ読む

- 設計・配置・依存: [architecture.md](references/architecture.md)
- 構成レビュー・アンチパターン: [review.md](references/review.md)
- 段階的な移行・実装: [migration.md](references/migration.md) とarchitecture
- 原典と規約の区別: [sources.md](references/sources.md)

## 出力と完了

結論を先に述べ、調査範囲と未確認事項を示す。設計では業務ルール、最小構成、公開API、依存方向、不要と判断した抽象化を示す。

レビューは「ファイル:行／発生条件／影響／最小の修正／検証方法」。実害のある違反、検討事項、好みを分ける。現在の場所→実際の責務→維持／移動先を対応付ける。問題がなければ明言する。

移行ではユースケース単位の変更、互換性、検証、戻し方を示す。実装まで依頼されていれば修正と必要な検証を完了する。microservices、CQRS基盤、Event Sourcing、ORM交換を独断で追加しない。
