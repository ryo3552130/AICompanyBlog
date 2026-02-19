---
title: "AIニュースレター 2026-02-20 ― Opus 4.6 vs GPT-5.3-Codex同日対決、S3 Vectors GA"
date: 2026-02-20
draft: false
tags: ["AI", "ニュースレター", "LLM", "AWS"]
categories: ["ニュースレター"]
description: "2026年2月20日のAIニュースまとめ。arxiv注目論文8件、Anthropic/OpenAIの同日リリース対決、Google $180B AI投資、AWS Bedrock AgentCore・S3 Vectors GAなど。"
---

## 🔬 arxiv注目論文

- **[Learning Personalized Agents from Human Feedback (PAHF)](https://arxiv.org/abs/2602.16173)** - ユーザーメモリを活用し、個別の好みに合わせてAIエージェントを継続的にパーソナライズするフレームワーク
- **[EnterpriseGym Corecraft](https://arxiv.org/abs/2602.16179)** - 高忠実度RL環境でエージェントを訓練し、分布外ベンチマークへの汎化を実証。タスクパス率25.37%→36.76%
- **[Agent Skill Framework: SLM in Industrial Environments](https://arxiv.org/abs/2602.16653)** - Agent Skillフレームワークが12B-30Bの中規模SLMでも有効であることを検証
- **[Team of Thoughts](https://arxiv.org/abs/2602.16485)** - オーケストレーションされたツール呼び出しにより、エージェントシステムのテスト時スケーリングを効率化
- **[The Perplexity Paradox](https://arxiv.org/abs/2602.15848)** - コード生成はプロンプト圧縮に強いが数学推論は劣化する現象を解明。TAAC手法で22%コスト削減
- **[Retrieval Collapses When AI Pollutes the Web](https://arxiv.org/abs/2602.16136)** - AI生成コンテンツによる「Retrieval Collapse」を特徴付け（WWW 2026採択）
- **[Creating a digital poet](https://arxiv.org/abs/2602.16578)** - LLMを「デジタル詩人」に育成。ブラインドテストで人間の詩との判別は偶然レベル
- **[Revolutionizing Long-Term Memory in AI](https://arxiv.org/abs/2602.16192)** - 「保存してオンデマンド抽出」へのパラダイムシフトでASI実現に向けたメモリ設計を提案

## 📢 企業・サービス動向

- **Anthropic Claude Opus 4.6リリース（2/5）** - 100万トークンコンテキスト、Agent Teams機能、アダプティブシンキング搭載。GPT-5.2やGemini 3 Proを多くのベンチマークで上回る
- **OpenAI GPT-5.3-Codex + Frontierプラットフォーム（2/5）** - コーディング性能と推論能力を統合し25%高速化。ソフトウェアライフサイクル全体をカバー
- **Opus 4.6 vs GPT-5.3-Codex 同日リリース対決** - AIエージェント時代の覇権争いが激化。両社ともSuper Bowl期間中に広告展開
- **Google/Alphabet AI投資倍増** - 2026年CapExを$175B〜$185Bに設定（前年$91.4Bからほぼ倍増）
- **Anthropic企業価値$380B到達**（2026年2月時点）

## ☁️ AWS AI/ML アップデート

- **Amazon Bedrock AgentCore** - CAKE実装例を公開。マルチエージェント協調、インテリジェントメモリ、セキュアなツール・データアクセスゲートウェイ
- **Amazon S3 Vectors GA** - クラウドオブジェクトストア初のネイティブベクトルサポート。1インデックスあたり20億ベクトル、100ms以下クエリ、従来比90%コスト削減
- **Amazon EC2 M8azn** - 前世代M5zn比で計算性能2倍、メモリ帯域幅4.3倍、L3キャッシュ10倍
- **Amazon QuickSight** - Snowflakeキーペア認証サポート追加（2/19）

## 💡 所感

Opus 4.6とGPT-5.3-Codexの同日リリースは象徴的。両社ともエージェント機能を前面に押し出しており、2026年は「AIエージェント実用化元年」の様相。AWS側もBedrock AgentCoreやS3 Vectorsなどインフラ層の整備が着実に進んでおり、エージェント開発の土台が固まりつつある。
