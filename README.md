## 【AWS インフラストラクチャ構築（Terraform）】
このリポジトリでは、Terraform を使用して AWS 上にインフラを構築（一部、手動）するためのコードを提供します。

### 📂 ディレクトリ構成

```bash
skill-typing-infra-Terraform/
├── README.md    # プロジェクトの概要や使い方を記載したドキュメント
├── main.tf      # Terraform のメイン設定ファイル（リソース定義）
```

### 🛠 必要なツール
このコードを実行する前に、以下のツールをインストールしてください。

- Terraform  
- AWS CLI

また、`aws configure` を実行し、AWS のクレデンシャルを設定してください。

```bash
aws configure
```

### ⚠️ 注意点
- **ACM 証明書の変更**

別の証明書を使用する場合は、ARN を設定してください。
```bash
# ACM 証明書の ARN（外部入力用）
variable "acm_certificate_arn" {
  default     = "arn:aws:acm:ap-northeast-1:xxx"
  description = "ARN of the ACM Certificate"
}
```

- **ECS コンテナイメージの変更**

自分の ECR リポジトリを使用する場合は、以下の箇所を変更してください。
```bash
image = "YOUR_ECR_REPO_URL"
```

- **RDS のパスワード変更**

RDS のパスワードは AWS Secrets Manager で管理されています。デフォルトでは `rds_password_v5` という名前のシークレットが作成されます。
別の名前に変更する場合は、以下のように設定してください。
```bash
resource "aws_secretsmanager_secret" "rds_password_custom" {
  name = "your-custom-rds-password"
}
```

### 🚀 デプロイ手順
**1. リポジトリのクローン**  
```bash
git clone https://github.com/Winter-Hackathon-2025-A-Team/skill-typing-infra-Terraform.git
cd skill-typing-infra-Terraform
```

**2. Terraform の初期化**

Terraform のプラグインをダウンロードし、初期化を行います。
```bash
terraform init
```

**3. リソースの適用**

Terraform を適用し、インフラをデプロイします。
```bash
terraform apply
```

**4. リソースの削除**

作成したリソースを削除する場合は、以下のコマンドを実行してください。
```bash
terraform destroy
```

### 📌 構築するインフラ構成図

![Screenshot 2025-02-21 at 7.29.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3662571/14c68cef-7863-4742-97de-dc012a8dfd49.png)

- **AWS プロバイダーの設定**  
  - AWS の東京リージョン (**ap-northeast-1**) でインフラを構築するよう設定。  

- **Secrets Manager から ACM 証明書の ARN を取得**  
  - SSL 証明書の **ARN を Secrets Manager から取得** し、HTTPS 通信に利用。  

- **ネットワークの構築**  
  - **VPC（仮想ネットワーク）**  
    - 10.0.0.0/16 の範囲で VPC を作成し、DNS を有効化。  
  - **パブリックサブネット**  
    - インターネットに直接接続可能なサブネット。  
    - **ap-northeast-1a** と **ap-northeast-1c** の 2 つの AZ に配置。  
  - **プライベートサブネット**  
    - インターネットに直接接続できないサブネット（NAT 経由で通信）。  
    - **ap-northeast-1a** と **ap-northeast-1c** に、それぞれ 2 つずつ作成。  
  - **インターネットゲートウェイ (IGW)**  
    - VPC をインターネットに接続させるためのゲートウェイ。  
  - **ルートテーブル**  
    - **パブリックサブネット** は IGW を使ってインターネットに接続。  
    - **プライベートサブネット** は NAT ゲートウェイ経由でインターネットに接続。  
  - **NAT ゲートウェイ**  
    - **プライベートサブネットの EC2 インスタンスや ECS タスク** が外部と通信できるようにする。  

- **ECS クラスターの作成**  
  - AWS のコンテナオーケストレーションサービス **ECS (Fargate)** を使用。  
  - **ECS クラスター (`my-ecs-cluster`) を作成**。  
  - ECS タスクが AWS サービスにアクセスするための **IAM ロール (`ecs-task-execution-role`) を作成**。  

- **ECS タスクの定義**  
  - **ECS タスク (Datadog) を定義し、Fargate で動作するよう設定。**  
  - 3 つのコンテナを実行：  
    - **Datadog エージェント**  
    - **CWS（Cloud Workload Security）**  
    - **アプリケーションコンテナ (`my-app-repo`)**  
  - タスクには以下の環境変数を設定：  
    - **MySQL の接続情報**  
    - **Cognito（認証情報）**  
    - **Datadog API キー**  
    - **OpenAI API キー** など。  

- **ECS サービスの設定**  
  - **ECS Fargate タスクをプライベートサブネットで実行**。  
  - **ロードバランサー (ALB) と接続** して外部からアクセス可能に。  

- **ALB（アプリケーションロードバランサー）の設定**  
  - **HTTPS 通信を許可するパブリック ALB を作成**。  
  - **443 番ポート (HTTPS) を開放**。  
  - **ターゲットグループを作成し、ECS のコンテナを登録**。  
  - **ヘルスチェックを設定し、正常なターゲットのみトラフィックを送信**。  

- **RDS（MySQL データベース）の設定**  
  - **プライベートサブネットに配置し、外部から直接アクセス不可**。  
  - **IAM Secrets Manager でパスワード管理**。  
  - **ECS タスクのセキュリティグループだけがアクセス可能**。  

- **CloudFront + S3 の設定**  
  - **S3 バケット (`my-cloudfront-bucket-tokyo`) を作成し、React アプリをホスティング**。  
  - **CloudFront (CDN) を設定し、S3 の静的コンテンツを配信**。  
  - **CloudFront OAI (オリジンアクセスアイデンティティ) を使い、S3 への直接アクセスを禁止**。  

- **Route 53（DNS）の設定**  
  - **Route 53 で CloudFront のエイリアスレコードを設定** し、**独自ドメイン (`honda333.blog`) でアクセス可能** に。  

### 💬 問い合わせ
質問や問題がある場合は、GitHub の Issues または Pull Request でお問い合わせください。
