以下の要件に基づき、プロダクション品質のAWSアーキテクチャ図を作成して

# アーキテクチャ要件

■ コンピュート
- Amazon ECS（Fargate）
- サービスは2つ：api / worker
- Service Auto Scaling 有効

■ ネットワーク
- 単一Availability Zone
- 1つのVPC
- Public Subnet
    - Application Load Balancer
    - NAT Gateway
- Private Subnet
    - ECSタスク
    - Amazon RDS（Single-AZ / Private）

■ トラフィックフロー
- Internet → ALB → ECS(api)
- ECS(worker)は外部公開しない
- ECS → RDS 接続
- ECS → NAT Gateway → Internet（外部API通信）

■ コンテナ/CI-CD
- GitHub Actions がDockerイメージをビルド
- Amazon ECR へPush
- ECSがECRからイメージをPull

■ 監視
- ECSログはCloudWatch Logsへ送信

# セキュリティ強化要件

■ エッジ保護
- Internet → AWS WAF → ALB の構成にする
- WAFはALBに関連付ける
- 不正アクセス防御を示す

■ TLS暗号化
- ACMで発行した証明書をALBにアタッチ
- HTTPS終端はALBで実施
- HTTP → HTTPS リダイレクトを明示

■ シークレット管理
- Secrets ManagerでDB認証情報を管理
- アプリケーションはIAMロール経由でSecrets取得
- 環境変数に平文保存しない構成

# 出力制約

- 以下は必ず表現する：
    - VPC
    - Public Subnet
    - Private Subnet
- 各AWSサービスは明確なラベルを付ける
- 可読性を重視したレイアウトにする
- 解説や説明文は出力しない
