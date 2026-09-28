# 一次資料と適用範囲

確認日: 2026-09-28。

- [Google: Gemini 3.1 Pro Preview](https://ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview): 正確なモデルIDと用途・対応機能。
- [Google: Models](https://ai.google.dev/gemini-api/docs/models): 採用時の候補確認。
- [Google: Pricing](https://ai.google.dev/gemini-api/docs/pricing): 現在の料金確認。
- [Google: Deprecations](https://ai.google.dev/gemini-api/docs/deprecations): 提供終了・移行確認。
- [Google: Structured outputs](https://ai.google.dev/gemini-api/docs/structured-output): schemaを指定した出力。schema適合と業務上の正しさは別に扱う。
- [Google: Function calling](https://ai.google.dev/gemini-api/docs/function-calling): tool呼び出しの生成と実行側の役割。
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents): 固定workflowと動的agentの区別、必要な複雑さだけを導入する考え方。

決定論的処理の判断表、重大度、開発フロー、評価基準はこれらを参考にした本スキルの設計指針。特定モデルの万能性や完全再現性を主張するものではない。LLM providerが変わっても、意味・権限・副作用をコードで検証する境界は保つ。
