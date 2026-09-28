# 最小のDDD構成

## 最初にモデルの意味を決める

「注文を確定する」「出荷済み注文は取り消せない」のように、業務の言葉と不変条件から始める。不明な仕様は暫定と明記する。同名の「顧客」でも請求とサポートで意味が違えば別のモデルでよい。bounded contextを1テーブル／1サービスと機械的に対応させない。[Bounded Context](https://martinfowler.com/bliki/BoundedContext.html)

## monorepoと単一アプリ

最初は1アプリ内で成立させる。以下は配置例であり、全ファイルを生成する雛形ではない。

```text
repo/
├── apps/
│   └── api/
│       ├── package.json
│       ├── src/
│       │   ├── domain/ordering/order.ts
│       │   ├── application/
│       │   │   ├── ordering/confirm-order.ts
│       │   │   └── ports/order-store.ts
│       │   ├── outbound/persistence/sql-order-store.ts
│       │   └── inbound/
│       │       ├── http/confirm-order-route.ts
│       │       └── bootstrap/main.ts
│       └── tests/
└── packages/                 # 本当に共有する機能ができたら追加
```

単一アプリなら `apps/api/` の内容をリポジトリ直下に置く。workspace導入は不要。

複数の実行アプリが同じ業務機能を使うなら、`packages/ordering/src/{domain,application,outbound,inbound}` へ境界単位で切り出せる。公開口は `application/public.ts`（ユースケースとDTO）、必要なら `inbound/bootstrap/public.ts`（組み立てfactory）に限定する。package exports等で内部を隠す。apps側のbootstrapからfactoryを呼び出す。exportだけのために `src/index.ts` を必須にしない。

- 横断的な `packages/domain`、`packages/application` という層別package分割から始めない。異なる業務の変更が同じ層に集まるため。
- `apps → packages` を基本とし、packageからapp、appから別appのソースを参照しない。
- context AからBを使う場合、AのapplicationポートをAのoutboundが実装し、Bの公開APIへ変換する。Bの集約やDBを直接共有しない。
- contractsを共有するなら境界を越えるDTOやイベントに限定し、全ドメインモデルの共通倉庫にしない。
- package境界、context境界、デプロイ単位は一致しなくてよい。分割理由を意味、所有者、利用者で説明する。

4ディレクトリ規約は業務コードが対象。UI専用packageや設定packageにdomainを強制しない。フレームワークが `src/app` 等を要求する場合は維持し、薄い入口から委譲する。

## モデルを小さく保つ

| 概念 | 導入条件 | 最小の実装 |
| --- | --- | --- |
| Entity | 同一性を保って状態が変わる | IDと意味のある操作。任意setterを公開しない |
| Value Object | 値の制約や比較に意味がある | 検証付き生成関数とimmutableな値 |
| Aggregate | 一緒に整合性を守る状態がある | root経由で変更する小さな境界 |
| Domain service | Entityに自然に属さない業務判断 | 必要な値を受け取る純粋関数から |
| Repository/store | 集約取得・保存の技術を分離したい | 使う操作だけ。全table用のCRUD基底class不要 |
| Domain event | 業務上の出来事を必要とする利用者がいる | 意味のある事実。イベントバス導入は別判断 |

classがないことを貧血モデルとしない。immutableなデータと純粋な状態遷移関数でもルールを閉じ込められる。問題は同じルールが入口やDB処理に散在し、別経路から破れること。

## 注文確定の責務

1. inboundで入力を検証し、認証結果をapplication入力へ変換する。
2. applicationで注文取得、操作権限確認、domainの確定操作、保存を調整する。
3. domainで空注文の禁止や状態遷移を判定する。HTTPとDBは知らない。
4. outboundでDB変換、条件付き保存、技術例外を内側の契約に変換する。
5. inboundで結果を公開レスポンスへ変換する。

認証の技術はinbound、ユースケース実行権限はapplication、業務上の所有権・承認資格はdomainにも置ける。CLIやジョブからでも必要な制約を守る。

時刻・ID・外部情報は必要ならapplicationがポート経由で取得し、domainに値で渡す。domainから環境変数、DI container、ネットワーク、ORMを参照しない。

## 整合性と参照系

集約は「一緒にロードできる全データ」ではなく不変条件から決める。同時更新を考慮し、保存時のバージョン比較、ロック、一意制約も検証する。事前のdomainチェックだけでは競合を防げない。

transaction範囲はapplicationで決め、実装はoutboundへ。複数storeが同じtransactionに参加する契約が必要。`runInTransaction` で囲んでも別connectionなら原子性はない。

一覧はqueryポート→outboundのSQL→DTOでよい。更新用集約へ無理に復元せず、権限とtenant条件は維持する。[永続化設計](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)

DB更新と外部送信は二つの関数呼び出しでは原子的にならない。保証が必要な場合だけoutboxや再試行・重複排除を設計する。課金等を含む処理全体の再試行で副作用を二重実行しない。

## 境界検証

既存lintやimport graphを優先する。alias、type-only import、再export、package exports、生成コードまで見る。grepは候補抽出であり証明ではない。

domain→外層、application→adapter、通常inbound→outbound、outbound→inbound、context内部の横断参照、package循環を調べる。bootstrapの例外は指定範囲の配線だけに限定する。

## srcとscriptsの役割

拡張子や「手動で実行するか」ではなく、責務で配置を決める。

| 実行するもの | 配置の基本 | 理由 |
| --- | --- | --- |
| 注文取込、請求確定、業務データの再計算 | `src/inbound/cli` または `src/inbound/jobs` | 業務ユースケースを呼ぶ入口 |
| 引数解釈、結果表示、exit codeへの変換 | 同じinbound | CLI固有の変換 |
| DB client生成、環境設定、依存の注入 | `src/inbound/bootstrap` | 起動と配線 |
| build、codegen、開発環境の準備 | repoまたはpackage直下の `scripts/` | 開発・運用ツール |
| schema migrationファイル | ORM等の規定場所、またはoutbound配下 | 永続化構造の変更 |
| schema migrationを起動するwrapper | `scripts/` または既存deploy基盤 | 運用タスクの起動 |

`inbound/scripts` も責務が明確なら使えるが、新規では `cli`／`jobs` を基本として入口の種類を示す。ルートのscriptsに業務判断がある場合、入口はinbound、判断はdomain、処理順序はapplication、DBアクセスはoutboundへ分ける。ファイルをinboundへ移すだけでは責務分離にならない。

通常のCLI handlerはapplicationの公開APIを受け取り、outboundをimportしない。実行エントリはbootstrapに置き、adapterとユースケースを組み立ててCLI handlerへ渡す。handlerからbootstrapをimportして配線を逆流させない。

一度限りのbackfillも、業務ルールを守る必要があるならユースケースを通す。技術的なデータ変換はmigrationとして扱えるが、DB制約、再開位置、重複実行、既存業務ルールへの影響を検証する。cronやCIから動くという理由で権限・整合性チェックを迂回しない。

公開する実行コマンドと内部の実装言語は別に決める。Node系なら `pnpm run <task>` を共通入口にし、必要なshはその裏側に置ける。pnpm scriptsに長い業務ロジックを書かず、applicationやdomainからルートscriptsをimportしない。
