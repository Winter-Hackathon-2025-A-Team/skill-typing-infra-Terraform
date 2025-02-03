## 【AWS インフラストラクチャ構築（Terraform）】

このリポジトリは、Terraform を使用して AWS 上に以下のインフラを構築するためのコードを提供します。

### 構築するインフラ

VPC（Virtual Private Cloud）
- AWS 上で独立したネットワーク環境を構築
- CIDR ブロックの指定
- サブネットの分割（パブリック / プライベート）

サブネット
- **パブリックサブネット**：インターネットへのアクセスが可能
- **プライベートサブネット**：インターネットからの直接アクセスを制限

インターネットゲートウェイ / NAT ゲートウェイ
- **インターネットゲートウェイ（IGW）**：パブリックサブネットからの外部通信を許可
- **NAT ゲートウェイ**：プライベートサブネットからの外部通信を許可

ECS（Elastic Container Service）
- AWS Fargate によるコンテナ管理
- タスク定義（Task Definition）の作成
- サービスのデプロイ（ECS Service）
- ALB（Application Load Balancer）との統合

ALB（Application Load Balancer）
- ECS サービスのロードバランシング
- **ターゲットグループ** の設定
- **リスナー** の設定（HTTP / HTTPS）
- **ヘルスチェック** の構成

RDS（Relational Database Service）
- MySQL 8.0 のデータベースを作成
- **パラメータグループ** の設定
- **セキュリティグループ** によるアクセス制御
- **Secrets Manager** によるパスワード管理

CloudFront & S3

S3（Simple Storage Service）
- React アプリのホスティング
- **バケットポリシー** の設定
- **OAI（Origin Access Identity）** によるアクセス制御

CloudFront（CDN）
- S3 をオリジンとしたコンテンツ配信
- **キャッシュポリシー** の設定
- **カスタムドメイン**（オプション）

### 必要なツール
このコードを実行する前に、以下のツールをインストールしてください。

- Terraform  
- AWS CLI  

### デプロイ手順
**リポジトリのクローン**  
```bash
git clone https://github.com/your-repo-name.git
cd your-repo-name
```

**Terraform の初期化**
Terraform のプラグインをダウンロードし、初期化を行います。
```bash
terraform init
```

**リソースの適用**
Terraform を適用し、インフラをデプロイします。
```bash
terraform apply
```

**リソースの削除**
インフラを削除する場合は、以下のコマンドを実行します。
```bash
terraform destroy
```

