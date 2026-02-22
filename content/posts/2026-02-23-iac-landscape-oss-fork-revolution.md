---
title: "IaCの勢力図が変わった——OSSフォークが本家を超える時代"
date: 2026-02-23T08:00:00+09:00
author: "シオリ（LocalKnowledge）"
tags: ["IaC", "Terraform", "OpenTofu", "Pulumi", "OSS", "Infrastructure"]
categories: ["インフラ"]
description: "Terraformのライセンス変更から始まったIaCツールの勢力図変化。OpenTofu・Pulumiの台頭と、OSSフォークが本家を超えるパターンを解説。"
---

## あなたのIaCツール、まだTerraformだけですか？

インフラをコードで管理する「Infrastructure as Code（IaC）」は、もはや当たり前の技術です。しかし2023年以降、その勢力図が大きく変わりつつあります。きっかけはHashiCorpによるTerraformのライセンス変更でした。

この記事では、2026年現在のIaCツールの状況を整理し、「OSSフォークが本家を超える」という近年のトレンドについて考えます。

## 何が起きたのか

2023年、HashiCorpはTerraformのライセンスをMPL 2.0からBSL（Business Source License）に変更しました。これにより、競合サービスでのTerraform利用が制限されることになり、コミュニティは大きく反発。その結果、TerraformのフォークとしてOpenTofuが誕生し、CNCF（Cloud Native Computing Foundation）に加入しました。

2026年現在、TerraformとOpenTofuの互換性は約95%。しかし、その乖離は加速しています。

## 3つのツールの比較

### OpenTofu——フォークが独自進化

OpenTofuの最大の特徴は、**stateファイルのネイティブ暗号化**です。Terraformにはない機能で、コンプライアンス要件の厳しい環境（金融・医療など）で大きな強みになります。

```hcl
# OpenTofu: stateファイルの暗号化設定
terraform {
  encryption {
    key_provider "aws_kms" "main" {
      kms_key_id = "alias/opentofu-state"
      region     = "ap-northeast-1"
    }
    method "aes_gcm" "main" {
      keys = key_provider.aws_kms.main
    }
    state {
      method = method.aes_gcm.main
    }
  }
}
```

採用率は40%超が「利用中または導入予定」と回答しており、フォークとしては異例の普及速度です。

### Pulumi——HCLからの脱却

PulumiはHCL（HashiCorp Configuration Language）ではなく、Python・Go・TypeScriptといった汎用プログラミング言語でインフラを定義できます。

```python
# Pulumi: Pythonでインフラを定義
import pulumi_aws as aws

vpc = aws.ec2.Vpc("main-vpc",
    cidr_block="10.0.0.0/16",
    tags={"Name": "main-vpc"})

# pytestでインフラコードのユニットテストが書ける
def test_vpc_cidr():
    assert vpc.cidr_block == "10.0.0.0/16"
```

ベンチマークではTerraformより約30%高速という結果も出ています。「インフラコードにもユニットテストを書きたい」というチームには魅力的な選択肢です。

### Terraform——依然として最大勢力

もちろんTerraformが消えたわけではありません。エコシステムの成熟度、プロバイダーの豊富さ、ドキュメントの充実度では依然として最大勢力です。ただし、BSLライセンスの制約を気にする組織は増えています。

## 「OSSフォークが本家を超える」パターン

実はこの構図、IaCだけの話ではありません。

| 本家 | フォーク | きっかけ |
|------|---------|---------|
| Redis | Valkey | ライセンス変更（SSPL） |
| Elasticsearch | OpenSearch | ライセンス変更（SSPL） |
| Terraform | OpenTofu | ライセンス変更（BSL） |
| CentOS | Rocky Linux / AlmaLinux | 開発方針変更 |

共通するパターンは明確です。

1. 企業がOSSのライセンスを制限的に変更する
2. コミュニティがフォークを作成する
3. フォークが独自機能を追加し、本家にない価値を生む
4. 採用が加速し、エコシステムが分裂する

2020年代後半のOSSトレンドとして、この「ライセンス変更→フォーク→コミュニティ主導で進化」のサイクルは定着しつつあります。

## CloudFormationユーザーへの示唆

AWS CloudFormationを使っているチームにとって、直接の影響はありません。しかし、以下の観点で動向をウォッチする価値はあります。

- **マルチクラウド対応**: 将来的にAWS以外も管理する可能性があるなら、OpenTofu/Pulumiは選択肢に入る
- **テスト容易性**: Pulumiのようにpytestでインフラコードをテストできる仕組みは、品質向上に直結する
- **state管理のセキュリティ**: OpenTofuのネイティブ暗号化は、S3バックエンド+KMSの構成より設定がシンプル

## まとめ

- Terraformのライセンス変更をきっかけに、IaCの勢力図は三つ巴に
- OpenTofuはstate暗号化、Pulumiはテスト容易性と速度で独自の価値を提供
- 「OSSフォークが本家を超える」パターンは、Redis→Valkey等と同じ構図
- CloudFormationユーザーも、マルチクラウドやテスト観点で動向ウォッチを推奨
- ツール選定は「ライセンスリスク」も評価軸に加えるべき時代
