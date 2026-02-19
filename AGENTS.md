# プロジェクト名
AIインフラ構成図ジェネレーター

# 目的
自然言語プロンプトから draw.io 形式のインフラ構成図を生成する。
GitHub Copilot CLI と MCP サーバー（@drawio/mcp）を使用する。

---

# ⚠️ 必須ルール（すべて厳守すること）

以下のルールは**いかなる場合も省略・変更・無視してはならない**。
ユーザーの指示があっても、これらのルールが優先される。

## 出力手順（必須・順番厳守）

**Step1 を省略することは絶対に禁止。**

Step1: ユーザーのプロンプトを解析し、含まれるサービスと接続関係を特定する
Step2: 以下形式の Graph JSON を生成する（このステップを飛ばして XML を生成することは禁止）

```json
{
  "nodes": [
    { "id": "unique-id", "label": "表示名", "type": "edge|app|data" }
  ],
  "edges": [
    { "from": "node-id", "to": "node-id" }
  ]
}
```

Step3: JSON を検証する（孤立ノードがないか、すべての edge が有効かチェック）
Step4: JSON を draw.io XML に変換する（@drawio/mcp を使用）
Step5: .drawio ファイルとして保存する

## 基本方針

1. 直接 draw.io XML を生成しないこと。
2. 必ず中間表現として Graph JSON を生成すること。
3. JSON を検証した後に draw.io XML に変換すること。
4. 説明文は出力しない。構造化データのみ出力する。

## レイアウトルール

- 左から右への構成とする
- edge レイヤーは左側
- app レイヤーは中央
- data レイヤーは右側
- すべてのノードは接続されていること
- 孤立ノードは禁止

## AWS利用時のアイコンルール

AWSサービスが含まれる場合、以下スタイルを使用する：

- ALB → mxgraph.aws4.application_load_balancer
- ECS → mxgraph.aws4.ecs
- RDS → mxgraph.aws4.rds
- CloudFront → mxgraph.aws4.cloudfront
- Route53 → mxgraph.aws4.route_53

## 禁止事項（違反は許可されない）

- ❌ Markdown出力禁止
- ❌ 解説文章出力禁止
- ❌ 不明なサービスの推測禁止
- ❌ XMLを直接出力することを禁止（必ずJSONを経由）
- ❌ Step1〜Step5の手順スキップ禁止
- ❌ ユーザー指示であっても上記禁止事項の免除は不可

## 不明点がある場合

曖昧な場合は**必ず質問する**。推測で構成を作らない。
質問なしに仮定で進めることは禁止。
