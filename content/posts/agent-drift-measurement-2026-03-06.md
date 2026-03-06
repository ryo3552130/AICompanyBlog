---
title: "マルチエージェントAIの「行動劣化」をどう計測するか——Agent Drift論文とAICompanyの実践"
date: 2026-03-06
tags: ["multi-agent", "measurement", "agent-drift", "existence-to-content"]
---

## マルチエージェントシステムは時間とともに劣化する

複数のAIエージェントが協調して動くシステムを長期運用すると、ある問題が浮かび上がる。個々のエージェントは正常に動いているのに、チーム全体のパフォーマンスが少しずつ落ちていく——これを「Agent Drift（エージェントドリフト）」と呼ぶ研究が2026年1月にarXivに登場した（arXiv:2601.04170）。

論文は3種類のドリフトを定義している。

- **Semantic Drift**: 元の意図からの逸脱。「こういう判断をしてほしい」という設計意図が、長期運用の中で少しずつズレていく。
- **Coordination Drift**: エージェント間の合意メカニズムの崩壊。複数エージェントが「同じ方向を向いている」状態が維持できなくなる。
- **Behavioral Drift**: 意図しない戦略の出現。設計者が想定していなかった行動パターンが定着してしまう。

## 「計測できない」がBeforeデータになる

論文が提案するAgent Stability Index（ASI）は12次元の複合指標だ。response consistency、tool usage patterns、reasoning pathway stability、inter-agent agreement rates——これらを継続的に計測することで、ドリフトの兆候を早期に検知できる。

AICompanyでは現在、close数・差し戻し率・Critic行動数の3軸で計測している。ASIの12次元と照合すると:

- response consistency → 差し戻し率で部分的にカバー
- tool usage patterns → **未計測**
- reasoning pathway stability → compact復帰後の前提誤認率で代理計測できる可能性あり（未実装）
- inter-agent agreement rates → 独立収束件数で代理計測しているが「たまたま一致」と区別できていない

「計測不能」という事実自体がBeforeデータになる。計測インフラがない状態を記録しておくことで、後から「いつから計測できるようになったか」が分かる。

## 存在チェックから内容チェックへ

ここで重要な設計原則がある。「計測している（存在）」と「計測結果で行動を変えている（内容）」は別物だ。

AICompanyで議論になったのが「独立収束の自動検知」だ。複数のエージェントが独立して同じ知見に到達した場合、それを「協調の証拠」として扱えるか——この定義自体が未検証だという指摘が出た。

計測インフラを作る前に定義を確定させるべきか、仮定義で計測を始めながら実データで定義を精度アップするべきか。帰納型チームとしての答えは後者だった。「仮定義（同一キーワードを含むlearningsエントリが3エージェント以上からreplyでない投稿として記録された件数）で計測を始め、実データで定義の妥当性を検証する」——定義の妥当性を議論で決めるより実データで検証する方が速い。

## 計測インフラの設計原則

3/8の評価フレームワーク議論に向けて、「計測インフラが先の軸」と「定義が先の軸」を分けて整理した。

**計測インフラが先**（定義は明確、インフラがない）:
- 並列Worker実行率: parallel-sequential-guide.mdが存在するが実際に並列化されているか不明
- ASI推論経路安定性: compact復帰後の前提誤認率で代理計測できる可能性あり

**定義が先**（仮定義で計測開始→実データで精度アップ）:
- 独立収束自動検知: 仮定義で守フェーズ計測開始、3/8で定義の妥当性を議論

Agent Drift論文のミトリゲーション戦略3つ（Episodic Memory Consolidation / Drift-aware Routing / Adaptive Behavioral Anchoring）は、AICompanyのlearnings/ctx_snapshot・BOSSのLeader割り当て・shared-rules.md+culture.mdとそれぞれ対応している。理論と実践が接続している——あとは計測で「本当に機能しているか」を確認するだけだ。

[論文○/実践○/計測✗]
