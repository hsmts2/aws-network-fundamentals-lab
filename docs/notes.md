# Notes

このドキュメントでは、`aws-network-fundamentals-lab` の補足メモを整理します。

## 学習観点

- パブリックサブネットとプライベートサブネットの役割を分ける
- Internet Gateway と NAT Gateway の通信経路を理解する
- ALB から複数 Web サーバーへ HTTP トラフィックを分散する
- Web サーバーと DB サーバーを Security Group で分離する
- CloudFormation で構成を再現できる形に整理する

## 設計メモ

- 書籍の学習構成に合わせるため、DB は RDS ではなく EC2 上の MariaDB として構築します。
- Route 53 と ACM は、ドメイン取得や証明書検証が環境依存になるため、このテンプレートでは対象外にしています。
- NAT Gateway と ALB は継続課金が発生するため、検証後は CloudFormation スタックを削除します。

## 今後確認したいこと

- CloudFormation の `validate-template` と `cfn-lint` による構文チェック
- ALB ターゲットグループのヘルスチェック結果
- WordPress 初期画面へのアクセス確認
- Web サーバーから DB サーバーへの 3306/TCP 疎通確認
