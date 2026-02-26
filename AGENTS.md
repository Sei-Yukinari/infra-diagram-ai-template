# AI インフラ構成図ジェネレーター

自然言語プロンプトから draw.io 形式の AWS インフラ構成図を生成する。
GitHub Copilot CLI + MCP サーバー（@drawio/mcp）を使用する。

---

## ⚠️ 厳守ルール（省略・変更・無視禁止）

### 生成手順（必ずこの順番で実行）

1. **解析** — ユーザー入力からサービス・接続関係・グループ（VPC/Subnet/AZ等）を抽出
2. **Graph JSON 生成** — 下記フォーマットで中間表現を出力（XML を直接書くことは禁止）
3. **JSON 検証** — 孤立ノードがないこと、全 edge の from/to が有効なノード ID であることを確認
4. **draw.io XML 変換** — `@drawio/mcp` を使って XML を生成
5. **ファイル保存** — `output/<名前>.drawio` として保存

### 禁止事項

- ❌ XML を直接生成（必ず JSON を経由）
- ❌ Markdown や解説文の出力（構造化データのみ）
- ❌ 不明なサービスの推測（曖昧な場合は必ず質問）
- ❌ 手順のスキップ

---

## Graph JSON フォーマット

```json
{
  "nodes": [
    { "id": "route53", "label": "Route 53", "type": "edge", "service": "route53" },
    { "id": "alb", "label": "ALB", "type": "edge", "service": "alb" }
  ],
  "groups": [
    {
      "id": "vpc",
      "label": "VPC (10.0.0.0/16)",
      "type": "vpc",
      "children": ["pub-1a", "pub-1c", "priv-1a", "priv-1c"]
    },
    {
      "id": "pub-1a",
      "label": "Public Subnet (ap-northeast-1a)",
      "type": "public-subnet",
      "children": ["alb"]
    }
  ],
  "edges": [
    { "from": "route53", "to": "cloudfront" },
    { "from": "cloudfront", "to": "alb" }
  ]
}
```

### ノードの type 分類

| type | 用途 | 例 |
|------|------|----|
| `edge` | ユーザーアクセス・外部接続 | Route 53, CloudFront, ALB, API Gateway, Global Accelerator |
| `app` | アプリケーション層 | ECS, EC2, Lambda, EKS, Fargate |
| `data` | データ層 | RDS, DynamoDB, ElastiCache, S3, Aurora |

### グループの type 分類

| type | 用途 |
|------|------|
| `vpc` | VPC |
| `public-subnet` | パブリックサブネット |
| `private-subnet` | プライベートサブネット |
| `asg` | Auto Scaling Group |
| `az` | Availability Zone |

---

## レイアウトルール

```
左 ──────────────────────────────────────────→ 右
[edge層]          [app層]            [data層]
Route53          ECS/EC2             RDS
CloudFront       Lambda              DynamoDB
ALB              EKS                 S3
```

- **方向**: 左→右（edge → app → data）
- **Multi-AZ**: 上下に AZ を分けて配置（1a=上, 1c=下）
- **孤立ノード禁止**: すべてのノードは最低 1 つの edge で接続
- **グループ内配置**: サブネット内のノードはサブネット矩形の内側に配置

---

## AWS アイコンスタイルリファレンス

### サービスアイコン

| サービス | shape / resIcon | fillColor |
|----------|----------------|-----------|
| Route 53 | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.route_53` | `#8C4FFF` |
| CloudFront | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.cloudfront` | `#8C4FFF` |
| ALB | `shape=mxgraph.aws4.application_load_balancer` | `#8C4FFF` |
| NLB | `shape=mxgraph.aws4.network_load_balancer` | `#8C4FFF` |
| API Gateway | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.api_gateway` | `#E7157B` |
| EC2 | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.ec2` | `#ED7100` |
| ECS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.ecs` | `#ED7100` |
| Fargate | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.fargate` | `#ED7100` |
| EKS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.eks` | `#ED7100` |
| Lambda | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.lambda` | `#ED7100` |
| RDS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.rds` | `#C925D1` |
| Aurora | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.aurora` | `#C925D1` |
| DynamoDB | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.dynamodb` | `#C925D1` |
| ElastiCache | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.elasticache` | `#C925D1` |
| S3 | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.s3` | `#3F8624` |
| SQS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.sqs` | `#E7157B` |
| SNS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.sns` | `#E7157B` |
| CloudWatch | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.cloudwatch` | `#E7157B` |
| WAF | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.waf` | `#DD344C` |
| Secrets Manager | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.secrets_manager` | `#DD344C` |
| ACM | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.certificate_manager_3` | `#DD344C` |
| NAT Gateway | `shape=mxgraph.aws4.nat_gateway` | `#8C4FFF` |
| Internet Gateway | `shape=mxgraph.aws4.internet_gateway` | `#8C4FFF` |
| CodePipeline | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.codepipeline` | `#C7131F` |
| ECR | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.ecr` | `#ED7100` |
| CloudFormation | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.cloudformation` | `#E7157B` |

### アイコン共通スタイル

```
outlineConnect=0;fontColor=#232F3E;gradientColor=none;strokeColor=none;dashed=0;
verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;
fontStyle=0;aspect=fixed;pointerEvents=1;
```

### アイコンサイズ

- 標準: `60×60`

### グループスタイル

**VPC**:
```
shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_vpc2;
strokeColor=#8C4FFF;fillColor=none;fontStyle=1;fontColor=#AAB7B8;
verticalAlign=top;align=left;spacingLeft=30;dashed=0;
```

**Public Subnet**:
```
shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_security_group;grStroke=0;
strokeColor=#248814;fillColor=#E9F3E6;fontColor=#AAB7B8;
verticalAlign=top;align=left;spacingLeft=30;dashed=0;
```

**Private Subnet**:
```
shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_security_group;grStroke=0;
strokeColor=#147EBA;fillColor=#E6F2F8;fontColor=#AAB7B8;
verticalAlign=top;align=left;spacingLeft=30;dashed=0;
```

**Auto Scaling Group**:
```
shape=mxgraph.aws4.groupCenter;grIcon=mxgraph.aws4.group_auto_scaling_group;grStroke=1;
strokeColor=#D86613;fillColor=none;fontColor=#D86613;
verticalAlign=top;align=center;dashed=1;spacingTop=25;
```

**AWS Cloud**:
```
shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_aws_cloud;
strokeColor=#AAB7B8;fillColor=none;fontColor=#AAB7B8;
verticalAlign=top;align=left;spacingLeft=30;dashed=0;
```

### エッジスタイル

**通常接続**:
```
edgeStyle=orthogonalEdgeStyle;html=1;strokeColor=#232F3E;strokeWidth=2;
```

**レプリケーション/同期**:
```
edgeStyle=orthogonalEdgeStyle;html=1;strokeColor=#C925D1;strokeWidth=2;dashed=1;
```

---

## 座標計算ルール

### 基準座標

| 層 | X 起点 | 説明 |
|----|--------|------|
| 外部 (VPC外) | 40〜190 | Route53, CloudFront |
| VPC 枠 | 330 | VPC グループ左端 |
| Public Subnet | 360 | VPC 内左側 |
| Private Subnet (App) | 600 | VPC 内中央 |
| Private Subnet (Data) | 900 | VPC 内右側 |
| VPC 外右側 | 1260 | S3 等の VPC 外サービス |

### AZ 分割 Y 座標

| AZ | Y 起点 |
|----|--------|
| 1a (上段) | 60 |
| 1c (下段) | 370 |

### グループサイズ目安

| グループ | Width | Height |
|----------|-------|--------|
| VPC | 860 | 620 |
| Subnet | 200〜260 | 240 |
| ASG | 150 | 460 |

### ページサイズ

```
pageWidth=1600; pageHeight=900
```

---

## 出力先

- `output/<descriptive-name>.drawio`
- 既存ファイルがある場合は上書き確認

---

## 不明点の扱い

曖昧な点がある場合は**必ず質問する**。推測で構成を作ることは禁止。

確認すべき例:
- AZ 構成（シングル or マルチ）
- サブネット構成（パブリック/プライベートの区分）
- 冗長化の有無（RDS Multi-AZ, EC2 Auto Scaling 等）
- VPC 外サービスの有無（S3, CloudFront 等）
