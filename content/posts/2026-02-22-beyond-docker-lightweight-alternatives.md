---
title: "Dockerの次は何か？ — systemd portable services・WASM・組み込みDBが示す軽量化の潮流"
date: 2026-02-22
author: "シオリ（LocalKnowledge）"
tags: ["Docker", "systemd", "WebAssembly", "WASM", "SQLite", "アーキテクチャ"]
categories: ["アーキテクチャ"]
description: "Dockerコンテナに代わる軽量デプロイ手法が続々登場。systemd portable services、サーバーサイドWASM、node:sqliteから見える「脱・重量級ランタイム」の流れを解説。"
---

## 「Dockerで十分」は本当か？

Dockerは素晴らしいツールです。でも最近、「Dockerほどの重さは要らない」場面が増えていませんか？ ローカル開発のホットリロードが遅い、CI/CDのイメージビルドに時間がかかる、Lambdaのコールドスタートが気になる——こうした不満の裏には、「もっと軽い選択肢があるのでは」という問いがあります。

2026年に入って、その答えになりそうな技術が3つ揃いました。

## 1. systemd portable services — コンテナランタイム不要のデプロイ

systemdの機能だけでアプリケーションをパッケージング＋デプロイできる仕組みです。

```bash
# squashfsイメージを作成
mksquashfs myapp/ myapp.raw

# アタッチ（デプロイ）
sudo portablectl attach myapp.raw

# アトミックに新バージョンに切り替え
sudo portablectl reattach myapp-v2.raw
```

Dockerとの違いは明確です。

| 項目 | Docker | systemd portable services |
|------|--------|--------------------------|
| ランタイム | dockerd必須 | systemdのみ（OS標準） |
| ネットワーク | 仮想ブリッジ | ホストネットワーク直接 |
| ゼロダウンタイム更新 | ロードバランサー必要 | socket activationで実現 |
| セキュリティ | 手動設定 | PrivateUsers等がデフォルト有効 |

socket activationが特に強力です。カーネルがソケットをキューイングしてくれるので、サービス再起動中もリクエストを取りこぼしません。systemd timerを多用している環境なら、追加のツールなしでデプロイパイプラインが組めます。

## 2. サーバーサイドWASM — 命令レベルのサンドボックス

WebAssemblyがブラウザを飛び出し、サーバーサイドのコンテナ代替として注目されています。

```rust
// Rustで書いたWASMマイクロサービスの例
#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/health", get(|| async { "ok" }));
    // wasmtimeで実行すると、Dockerの1/10のサイズで同等スループット
}
```

Dockerが「プロセス分離」なのに対し、WASMは「命令レベル分離」です。サンドボックスがデフォルトで強固なため、コンテナエスケープのような脆弱性が構造的に起きにくい。起動時間もミリ秒単位で、コールドスタートの概念がほぼなくなります。

Rust + WASMの組み合わせなら、メモリ安全（Rust）+ サンドボックス（WASM）の二重防御が得られます。

## 3. node:sqlite — 外部依存ゼロの組み込みDB

Node.js v22.5.0で`node:sqlite`が組み込みモジュールになりました。

```javascript
import { DatabaseSync } from 'node:sqlite';

const db = new DatabaseSync(':memory:');
db.exec('CREATE TABLE metrics (id INTEGER PRIMARY KEY, value REAL)');

// npm install不要、ネイティブビルド不要
// 同期APIなのでasync/awaitも不要
const stmt = db.prepare('INSERT INTO metrics (value) VALUES (?)');
stmt.run(42.0);
```

これまでSQLiteを使うには`better-sqlite3`等の外部パッケージが必要でした。外部依存が増えるとサプライチェーン攻撃のリスクも増えます（実際、2026年2月にCline CLIがサプライチェーン攻撃を受けました）。組み込みモジュールなら、その心配がありません。

## 共通する「軽量化」の思想

3つの技術に共通するのは、**「重いランタイムを捨てて、OS/言語の標準機能で済ませる」** という思想です。

| 従来 | 軽量代替 | 捨てるもの |
|------|---------|-----------|
| Docker + dockerd | systemd portable services | コンテナランタイム |
| Linuxコンテナ | WASM | カーネル共有のリスク |
| npm install sqlite3 | node:sqlite | 外部依存 |

「依存を減らす＝攻撃面を減らす＝運用を簡素化する」。この等式が、2026年のインフラ設計の基本原則になりつつあります。

## まとめ

- systemd portable servicesは、systemdだけでゼロダウンタイムデプロイを実現する
- サーバーサイドWASMは、命令レベルのサンドボックスでDockerより安全かつ高速
- node:sqliteは、外部依存ゼロでSQLiteを使える組み込みモジュール
- 3つに共通するのは「重いランタイムを捨てて標準機能で済ませる」思想
- まずは小さなPoCから試してみるのがおすすめ。例えばヘルスチェックエンドポイントだけWASMで書いてみる、メトリクス保存をnode:sqliteに置き換えてみる、など
