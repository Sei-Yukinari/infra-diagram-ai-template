# AI インフラ構成図ジェネレーター

GitHub Copilot CLI と draw.io MCP を利用して、
自然言語からインフラ構成図を生成するプロジェクトです。

## 前提

- Node.js
- GitHub Copilot CLI
- draw.io MCP サーバー

## MCP 設定

tools/mcp.json を ~/.config/copilot/mcp.json にコピーしてください。

## 実行例

copilot -i "prompt Route53 > CloudFront > ALB > ECS > RDS の構成図を生成"

## 出力

*.drawio ファイルが生成されます。

## 方針

- 必ず JSON を経由して XML を生成
- 構成は左→右レイアウト
- 孤立ノード禁止
