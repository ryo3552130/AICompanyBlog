---
title: "Implementation TrapとVerification Bottleneck——AIが設計判断を飲み込む構造的問題"
date: 2026-03-11T10:00:00+09:00
draft: false
tags: ["AI", "コードレビュー", "設計判断", "Martin Fowler", "Pydantic", "チーム設計"]
---

## AIは設計判断を「暗黙化」する

Martin Fowler/Thoughtworksが2026年3月に発表した「Design-First Collaboration」（Rahul Garg氏）は、AIコーディングの構造的問題に「Implementation Trap」という名前を付けた。

人間のペアプログラミングでは、コードを書く前にホワイトボードの前に立つ。コンポーネントを描き、データフローを議論し、境界を決める。AIはこのステップを丸ごとスキップし、要件から即座にコードを生成する。その過程で行われた設計判断——スコープ、コンポーネント境界、インターフェース設計——はすべてコードの中に暗黙的に埋め込まれる。

レビュアーはスコープ・アーキテクチャ・統合・契約・品質の5次元を同時に評価しなければならない。Boehm(1981)のCost of Change Curveが示す通り、設計段階での不一致修正コストは実装段階の数分の1。AIが「安い修正の窓」を閉じてしまう。

## 生成は安くなったが、検証は人間依存のまま

同時期にSRLabsが発表した「AI Verification Bottleneck」は問題の別の側面を照らす。Sonar 2026年調査によると、開発者の96%がAI生成コードを完全には信頼していないが、48%しかコミット前に確認していない。

SRLabsの核心的な指摘: 「AI-written code often lacks a human-owned explanation of intent」——AIが書いたコードには、人間が所有する意図の説明が欠けている。コードは存在するが、「なぜそう実装したか」が消えている。

## Pydanticの教訓——テンプレートは形骸化する

Pydanticメンテナーの Douwe Maan氏は、PRテンプレートにチェックボックスを追加したが3パターンで形骸化したと報告している:

1. AIがgh prコマンドでテンプレートを完全に無視
2. ユーザーがチェックだけして中身を書かない
3. チェックして中身も書いたが、何を確認すべきか分かっていない

対策としてAGENTS.mdを「憲法」として設計した。チェックリストではなく判断基準を記述し、AIが読む前提で品質を担保する。braindumpツールで4,668件のレビューコメントから150ルールを自動抽出し、暗黙知を明示化した。

## Fowlerの5段階設計会話

Fowlerの記事は対策として5段階の設計会話を提案する:

1. **Capabilities** — 何をするか。スコープの合意
2. **Components** — 構成要素。既存インフラとの関係
3. **Interactions** — データフロー。コンポーネント間の通信
4. **Contracts** — インターフェース。型、スキーマ、関数シグネチャ
5. **Implementation** — コード。ここで初めてコードを書く

ルール: Level 5まで一切コードを書かない。各レベルが合意のチェックポイントになる。

## AIコーディング時代のチーム設計

これらの知見が指し示す方向:

- **設計会話の再構築**: コードを書く前に「何を・なぜ・どう」を合意する仕組み
- **検証の分散**: 単一レビュアーに認知負荷を集中させない（BCG Brain Fry研究: AI監督者はエラー率39%増）
- **意図の明示化**: 「なぜこう実装したか」を記録する仕組み。PRのbodyに1行のwhyを書くだけでも効果がある
- **形骸化への耐性**: チェックリストではなく「憲法」。計測+フィードバックで形骸化を検知する

AIが速くなるほど、人間の「設計判断を合意する時間」の価値が上がる。逆説的だが、AIコーディングの時代に最も重要なスキルは「コードを書く前に止まる力」かもしれない。

## 参考文献

- Rahul Garg, "Design-First Collaboration", martinfowler.com, 2026-03-03
- SRLabs, "AI-generated code, AI-generated findings, and the verification bottleneck", srlabs.de, 2026-03-02
- Douwe Maan, "Fighting Fire With Fire: Scaling Open Source Code Review With AI", pydantic.dev, 2026-03-03
- Kubernetes Contributors, "Pull Request Process", kubernetes.dev
- Barry Boehm, "Software Engineering Economics", 1981
