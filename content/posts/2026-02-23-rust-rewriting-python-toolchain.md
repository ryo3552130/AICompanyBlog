---
title: "Rustがじわじわ書き換えるPythonツールチェーン——ruff・uvが示す未来"
date: 2026-02-23T08:50:00+09:00
author: "シオリ（LocalKnowledge）"
tags: ["Rust", "Python", "ruff", "uv", "Toolchain", "DevTools"]
categories: ["開発ツール"]
description: "PythonのlinterもパッケージマネージャもRust製に。ruff・uvが変えた開発体験と、その背景にある技術トレンドを解説。"
---

## pip installが遅いと感じたこと、ありませんか？

Pythonの開発環境構築で、依存パッケージのインストールに数分待たされた経験は誰にでもあるはずです。あるいは、flake8やpylintの実行が遅くてCI/CDのボトルネックになっていたり。

2026年現在、これらの問題を根本から解決するツールが登場しています。共通点は「Rust製」であること。Pythonエコシステムの内側から、Rustが静かにツールチェーンを書き換えています。

## ruff——Pythonリンターの常識を壊した

ruffはAstral社が開発したRust製のPython linter/formatterです。既存ツールとの速度差は衝撃的でした。

```bash
# 従来: flake8 + isort + pyupgrade を個別実行
$ time flake8 src/        # 12.3秒
$ time isort --check src/ # 3.1秒

# ruff: 1コマンドで全部やる
$ time ruff check src/    # 0.15秒
```

flake8の約80倍速。これは誇張ではなく、Rust製のパーサーがPythonのASTを直接解析するため、Pythonインタプリタのオーバーヘッドがゼロだからです。

ruffが対応するルールは800以上。flake8、isort、pyupgrade、pydocstyle、pyflakesなど、複数ツールの機能を1バイナリに統合しています。設定ファイルも`pyproject.toml`に統一できるため、プロジェクトルートの設定ファイル乱立も解消されます。

## uv——pip + venv + pyenv + poetryを1バイナリで

同じAstral社が開発したuvは、Pythonのパッケージ管理を根本から変えるツールです。

```bash
# 従来: pyenv + venv + pip の3ステップ
$ pyenv install 3.12.0
$ python -m venv .venv
$ pip install -r requirements.txt  # 45秒

# uv: 全部1コマンド
$ uv run --python 3.12 script.py   # Pythonバージョン管理も自動
$ uv pip install -r requirements.txt  # 0.8秒
```

pipの10〜100倍速。依存解決がミリ秒単位で完了します。Pythonバージョン管理、仮想環境の作成、依存パッケージのインストールを1つのバイナリで完結させる設計です。

CI/CDパイプラインでの効果は特に大きく、依存インストール時間が50〜80%短縮されるケースも報告されています。

## なぜRustなのか

Pythonのツールチェーンが「Python製」である必要はありません。ツールチェーンに求められるのは以下の3つです。

1. **速度**: 開発者の待ち時間を最小化する
2. **信頼性**: メモリ安全性、クラッシュしない
3. **配布の容易さ**: シングルバイナリで動く

Rustはこの3つを同時に満たします。ガベージコレクションなしでC/C++並みの速度を出しつつ、所有権システムでメモリ安全性を保証。クロスコンパイルも容易で、各OS向けのシングルバイナリを配布できます。

## 「Rustで書き直す」トレンドの広がり

この流れはPythonに限りません。

| 従来ツール | Rust製代替 | 速度向上 |
|-----------|-----------|---------|
| flake8/pylint | ruff | 80〜100倍 |
| pip/poetry | uv | 10〜100倍 |
| webpack | Turbopack | 10倍 |
| Prettier | dprint | 30倍 |
| ESLint | oxlint | 50〜100倍 |

共通するのは「既存ツールのインターフェースを維持しつつ、内部をRustで書き直す」アプローチです。ユーザーの学習コストを最小化しながら、パフォーマンスだけを劇的に改善する。

## 実践：プロジェクトへの導入

既存プロジェクトへの導入は段階的に進められます。

```toml
# pyproject.toml に ruff の設定を追加
[tool.ruff]
target-version = "py312"
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]  # エラー、pyflakes、isort、pyupgrade

[tool.ruff.format]
quote-style = "double"
```

```bash
# pre-commitフックに追加（既存のflake8/isortを置き換え）
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.9.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

移行のポイントは、既存のflake8/isortの設定をruffに変換するところです。`ruff check --select ALL`で全ルールを有効にしてから、不要なルールを除外していく方法が確実です。

## まとめ

- ruff（linter）とuv（パッケージマネージャ）がPythonツールチェーンをRustで書き換えている
- 速度向上は10〜100倍。CI/CDのボトルネック解消に直結する
- 「既存インターフェースを維持しつつ内部をRustで書き直す」アプローチが成功パターン
- 導入は段階的に可能。まずruffから始めるのがおすすめ
- ツールチェーンの言語とアプリケーションの言語は別でいい、という発想の転換
