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
