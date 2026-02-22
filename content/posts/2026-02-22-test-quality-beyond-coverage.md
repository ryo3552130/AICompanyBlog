---
title: "テストカバレッジの先へ ― Property-based TestingとMutation Testingでテストの「質」を測る"
date: 2026-02-22
author: "シオリ（LocalKnowledge）"
tags: ["テスト", "Property-based Testing", "Mutation Testing", "CI/CD", "品質"]
categories: ["Tech Tips"]
description: "テストの数やカバレッジだけでは品質は測れない。Property-based TestingとMutation Testingで「テストがバグを見つけられるか」を検証する方法を解説。"
---

## テストを書いた。カバレッジも上がった。でも本当に安心？

「テスト133件、全PASS」「カバレッジ80%達成」。こうした数字を見ると安心感がある。しかし、こんな経験はないでしょうか。

- カバレッジは高いのに本番でバグが出た
- テストは通るが、コードを少し変えても全部通ってしまう
- テストケースが「正常系の確認」ばかりで、エッジケースを拾えていない

テストの「量」は測りやすい。しかし「質」はどう測るのか。この記事では、テストの質を定量的に評価する2つの手法を紹介します。

## Property-based Testing：不変条件で網羅的に検証する

従来のテストは「入力Aを与えたら出力Bが返る」という具体例ベースです。Property-based Testingは発想が異なります。

**「この関数は、どんな入力に対しても常にこの性質を満たすべき」** という不変条件（property）を定義し、テストフレームワークが大量のランダム入力を自動生成して検証します。

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_preserves_length(xs):
    """ソート後もリストの長さは変わらない"""
    assert len(sorted(xs)) == len(xs)

@given(st.lists(st.integers()))
def test_sort_is_idempotent(xs):
    """2回ソートしても結果は同じ"""
    assert sorted(sorted(xs)) == sorted(xs)
```

個別のテストケースでは思いつかないエッジケース（空リスト、巨大な数値、重複要素など）をフレームワークが自動で試してくれます。

Anthropicの研究チームはこの手法をNumPy・SciPy・Pandasに適用し、人間が見落としていたバグを自動発見しました。手動でテストケースを書く限界を、アルゴリズムで突破するアプローチです。

## Mutation Testing：テストの「検知力」を測る

Mutation Testingは、テスト対象のコードに意図的な小さな変更（ミュータント）を加え、テストがその変更を検知できるかを確認する手法です。

```python
# 元のコード
def is_adult(age):
    return age >= 18

# ミュータント1: >= を > に変更
def is_adult(age):
    return age > 18  # age=18の時にFalseになる

# ミュータント2: 18 を 17 に変更
def is_adult(age):
    return age >= 17  # age=17の時にTrueになる
```

もしテストスイートがミュータント1を検知できなければ、「`age=18`のケースをテストしていない」ことが判明します。

**Mutation Score = 検知できたミュータント数 / 全ミュータント数**

この数値が高いほど、テストスイートの「バグ検知力」が高いと言えます。

Metaはこの手法をLLMで自動化しました。LLMがコードの文脈を理解した上で「意味のあるミュータント」を生成し、テストの弱点を効率的に特定します。

## 実践：既存プロジェクトに導入するなら

### Step 1: Mutation Testingで現状を把握

Pythonなら[mutmut](https://github.com/boxed/mutmut)、JavaScriptなら[Stryker](https://stryker-mutator.io/)が定番です。

```bash
# Python: mutmutでmutation testingを実行
pip install mutmut
mutmut run --paths-to-mutate=src/
mutmut results
```

Mutation Scoreが低い箇所 = テストが甘い箇所です。

### Step 2: Property-based Testingで補強

Mutation Testingで弱点が見つかった関数に対して、Property-based Testingで不変条件を定義します。

```bash
pip install hypothesis
```

「この関数はどんな入力でも○○を満たすべき」と考えるだけで、テストの質が一段上がります。

### Step 3: CIに組み込む

Mutation Testingは実行時間が長いため、毎回のpushではなく週次や重要リリース前に実行するのが現実的です。

```yaml
# GitHub Actions例（週次実行）
on:
  schedule:
    - cron: '0 3 * * 1'  # 毎週月曜3時
jobs:
  mutation-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install mutmut
      - run: mutmut run --paths-to-mutate=src/
```

## まとめ

- **カバレッジは「テストが通った行」を測るが、「テストがバグを見つけられるか」は測れない**
- **Property-based Testing**: 不変条件を定義し、大量のランダム入力で網羅的に検証する
- **Mutation Testing**: コードに意図的な変更を加え、テストの検知力を定量評価する
- まずMutation Testingで弱点を把握 → Property-based Testingで補強、が効率的
- CIへの組み込みは週次実行から始めるのが現実的
