# aws-network-fundamentals-lab

「AWSネットワーク入門［改訂4版］」の学習内容を、AWS CloudFormation で再現するためのリポジトリです。

本リポジトリは、書籍で学習した AWS ネットワーク構成を、自分の理解に基づいて Infrastructure as Code として整理したものです。書籍内容を転載するものではなく、学習した構成を CloudFormation テンプレートとして再現・管理することを目的としています。

---

<br>

## 概要

本リポジトリでは、書籍で学習した AWS ネットワーク構成のうち、VPC、EC2、NAT Gateway、Application Load Balancer、Web サーバー、DB サーバー構成を CloudFormation テンプレートとして管理します。

手順ベースで作成した AWS リソースを IaC（Infrastructure as Code）として定義することで、構成の再現性、変更管理、レビュー、再デプロイを容易にすることを目的としています。

---

<br>

## 対象書籍と学習範囲

| 項目 | 内容 |
| --- | --- |
| 対象書籍 | AWSネットワーク入門［改訂4版］ |
| 学習テーマ | AWS 上でのネットワーク構築 |
| 主な学習対象 | VPC / EC2 / Internet Gateway / NAT Gateway / Security Group / Network ACL / ALB / Route 53 / VPC Endpoint / VPC Peering / Site-to-Site VPN |
| 本リポジトリで作成する範囲 | VPC / EC2 / NAT Gateway / Security Group / ALB / Apache / MariaDB / WordPress |
| 作成するAWS構成 | ALB + Web サーバー2台 + DB サーバーの構成 |
| IaC | AWS CloudFormation |
| テンプレート形式 | YAML |
| リージョン | `ap-northeast-1` |

---

<br>

## リポジトリの目的

このリポジトリの目的は、単に CloudFormation テンプレートを保存することではなく、以下の観点を整理することです。

* AWS ネットワーク構成をコードとして再現する
* 手動構築した内容を CloudFormation に置き換える
* VPC、サブネット、ルートテーブル、セキュリティグループの関係を整理する
* パブリックサブネットとプライベートサブネットの役割を理解する
* NAT Gateway によるプライベートサブネットからのアウトバウンド通信を理解する
* ALB による Web サーバーの負荷分散構成を理解する
* Web サーバーと DB サーバーを分離した基本的な構成を理解する
* GitHub 上で学習成果をポートフォリオとして管理する

---

<br>

## アーキテクチャ

以下は、本リポジトリの CloudFormation テンプレートで作成する AWS 構成図です。

<img src="./diagrams/aws-network-fundamentals-architecture.png" alt="Architecture" width="900">

---

<br>

## 作成される構成

```text
Internet
  |
  | HTTP:80
  v
Application Load Balancer
  |
  | HTTP:80
  v
Web Server EC2 x 2
  |
  | TCP:3306
  v
DB Server EC2
```

ネットワーク構成は以下です。

```text
VPC: 10.0.0.0/16
├── Public Subnet A: 10.0.0.0/24
│   ├── NAT Gateway
│   └── Web Server A
│
├── Public Subnet C: 10.0.10.0/24
│   └── Web Server C
│
└── Private Subnet A: 10.0.1.0/24
    └── DB Server
```

---

<br>

## 作成される主な AWS リソース

| Resource | 設定値 / 備考 |
| --- | --- |
| VPC | CIDR: `10.0.0.0/16` / DNS ホスト名: 有効 |
| パブリックサブネット A | `10.0.0.0/24` |
| パブリックサブネット C | `10.0.10.0/24` |
| プライベートサブネット A | `10.0.1.0/24` |
| Internet Gateway | VPC にアタッチ |
| NAT Gateway | パブリックサブネット A に配置 / Elastic IP を割り当て |
| ルートテーブル | パブリック用・プライベート用 |
| セキュリティグループ | ALB 用 / Web 用 / DB 用 |
| IAM ロール | EC2 用 IAM ロール / SSM Session Manager 対応 |
| EC2 Web サーバー A | Amazon Linux 2 / Apache / PHP / WordPress |
| EC2 Web サーバー C | Amazon Linux 2 / Apache / PHP / WordPress |
| EC2 DB サーバー | Amazon Linux 2 / MariaDB |
| Application Load Balancer | Internet-facing / HTTP:80 |
| Target Group | Web サーバー2台を登録 |
| Listener | HTTP:80 |

---

<br>

## 本テンプレートで対象外にしたもの

| 対象外 | 理由 |
| --- | --- |
| Route 53 | ドメイン取得やDNS設定が環境依存になるため |
| ACM | HTTPS化にはドメインと証明書検証が必要になるため |
| VPC Peering | 今回は単一VPCの構成に絞るため |
| Site-to-Site VPN | 個人検証環境では接続先ルーターの準備が難しいため |
| VPC Endpoint | 別テンプレートまたは追加検証として扱うため |
| RDS | 書籍の学習構成に合わせ、EC2 上の MariaDB として構築するため |

---

<br>

## ディレクトリ構成

```text
.
├── README.md
├── templates/
│   └── aws-network-fundamentals.yaml
├── parameters/
│   └── aws-network-fundamentals-lab.example.json
└── diagrams/
    └── aws-network-fundamentals-architecture.png
```

---

<br>

## CloudFormation テンプレート

CloudFormation テンプレートは以下に格納しています。

[templates/aws-network-fundamentals.yaml](templates/aws-network-fundamentals.yaml)

このテンプレートでは、VPC、サブネット、Internet Gateway、NAT Gateway、ルートテーブル、セキュリティグループ、IAM ロール、EC2 インスタンス、Application Load Balancer、Target Group などをまとめて作成します。

---

<br>

## パラメータファイル

パラメータファイルのサンプルは以下に格納しています。

[parameters/aws-network-fundamentals-lab.example.json](parameters/aws-network-fundamentals-lab.example.json)

実際にデプロイする場合は、サンプルファイルをコピーして使用します。

```bash
cp parameters/aws-network-fundamentals-lab.example.json parameters/aws-network-fundamentals-lab.json
```

`parameters/aws-network-fundamentals-lab.json` には、自分の環境に合わせた値を設定します。実環境用の値を含むため、GitHub にはコミットしない運用とします。

---

<br>

## 前提条件

以下の環境が必要です。

* AWS CLI v2
* AWS CLI の認証設定済み環境
* デプロイ先 AWS アカウント
* デプロイ先リージョン: `ap-northeast-1`
* 既存の EC2 キーペア
* CloudFormation、EC2、VPC、IAM、ELB 関連リソースを作成・更新・削除できる IAM 権限

---

<br>

## デプロイ・運用方法

### パラメータ準備

```bash
cp parameters/aws-network-fundamentals-lab.example.json parameters/aws-network-fundamentals-lab.json
```

### テンプレート検証

```bash
aws cloudformation validate-template \
  --template-body file://templates/aws-network-fundamentals.yaml
```

### スタック作成

```bash
aws cloudformation create-stack \
  --stack-name aws-network-fundamentals-lab \
  --template-body file://templates/aws-network-fundamentals.yaml \
  --parameters file://parameters/aws-network-fundamentals-lab.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-northeast-1
```

### スタック状態確認

```bash
aws cloudformation describe-stacks \
  --stack-name aws-network-fundamentals-lab \
  --region ap-northeast-1
```

### 動作確認

スタック作成後、CloudFormation の Outputs に表示される `AlbDnsName` を確認します。

```bash
aws cloudformation describe-stacks \
  --stack-name aws-network-fundamentals-lab \
  --query "Stacks[0].Outputs" \
  --region ap-northeast-1
```

`AlbDnsName` の URL にブラウザでアクセスし、WordPress の初期インストール画面が表示されれば成功です。

```text
http://<ALB_DNS_NAME>/
```

### スタック更新

```bash
aws cloudformation update-stack \
  --stack-name aws-network-fundamentals-lab \
  --template-body file://templates/aws-network-fundamentals.yaml \
  --parameters file://parameters/aws-network-fundamentals-lab.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-northeast-1
```

### スタック削除

```bash
aws cloudformation delete-stack \
  --stack-name aws-network-fundamentals-lab \
  --region ap-northeast-1
```

---

<br>

## 設計上の補足

WordPress 初期設定時は、任意のサイト名、管理ユーザー名、強力なパスワード、検証用メールアドレスを設定します。検証環境では検索エンジンにインデックスされないよう、Search engine visibility のチェックを推奨します。

---

<br>

## 書籍との差分

書籍の学習構成に合わせるため、DB は RDS ではなく EC2 上の MariaDB として構築します。Route 53、ACM、VPC Endpoint などは、ドメインや検証環境への依存が大きいため、このテンプレートでは対象外にしています。
