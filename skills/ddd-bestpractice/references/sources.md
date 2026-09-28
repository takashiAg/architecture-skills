# 原典と規約

確認日: 2026-09-28。原典の考え方を要約し、配置は本スキルの設計判断としている。

- [Cockburn: Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture): 外部をadapterで隔離し、applicationを独立して動作・検証する。
- [Fowler: Bounded Context](https://martinfowler.com/bliki/BoundedContext.html): モデルと言葉の境界、および境界間の関係。
- [Fowler: Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html): 業務ルールが外へ散らばるコスト。
- [Microsoft: DDD-oriented microservice](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice): 業務判断、処理の調整、永続化の責務分離。microservices採用の根拠にはしない。
- [Microsoft: Persistence layer](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design): 集約root単位の更新と参照系の別経路。
- [Fowler: Strangler Fig](https://martinfowler.com/bliki/StranglerFigApplication.html): 段階的な置換。
- [Fowler: Monolith First](https://martinfowler.com/bliki/MonolithFirst.html): 境界が不明な段階で分散のコストを先取りしない。

4ディレクトリ、applicationへのポート配置、inbound/bootstrapの配線例外は、このスキル固有の選択。DDDの唯一の正解として扱わない。
