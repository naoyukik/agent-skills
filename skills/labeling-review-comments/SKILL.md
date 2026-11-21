---
name: labeling-review-comments
description: Labeling review comments with Conventional Comments labels (praise, nitpick, suggestion, issue, todo, question, thought, chore, note). Fix urgency is communicated via blocking/non-blocking decorations, decided by a severity × occurrence-frequency matrix. Use when writing review comments, review reports, or PR/MR review feedback.
metadata:
  version: "2.2"
---

# レビューコメントラベリング（Conventional Comments）

レビューコメントに Conventional Comments 規格のラベルを付与する。
ラベルは指摘の種別を分類する。修正要否の度合いは blocking / non-blocking デコレーションで伝える。

一次情報は https://conventionalcomments.org/ とする。

## 1. フォーマット

```
<label> [decorations]: <subject>

[discussion]
```

- **label**: コメントの種別。1コメントにつき1つ
- **decorations**: 追加文脈。括弧で囲みカンマ区切りで並べる
- **subject**: コメントの主旨
- **discussion**: 補助的な文脈、理由、解決のための次の一手

機械可読性が規格の目的の一つです。フォーマットから外れないこと。

## 2. ラベル定義

主要9種を使う。

| ラベル | 意味 |
|---|---|
| **praise:** | 良い点を認める。1レビューにつき最低1件を目指す。虚偽の称賛は有害なので心からのものだけを書く |
| **nitpick:** | 些細な好みベースの要求。性質上 non-blocking とする |
| **suggestion:** | 現主題への改善提案。何を、なぜ改善するのかを明示する。blocking / non-blocking を付けて意図を伝える |
| **issue:** | 主題の具体的問題。ユーザー向けでも内部向けでもよい。suggestion との併記を推奨する。問題の有無が確定しないなら question を使う |
| **todo:** | 小さく些細だが必要な変更。issue / suggestion と区別することで読者の注意を向けやすくする |
| **question:** | 懸念の有無が不明確な場合の確認。著者への確認や調査の依頼が迅速な解決に繋がる |
| **thought:** | レビュー中に浮かんだアイデア。性質上 non-blocking とする。取り組みやメンタリングの機会に繋がる |
| **chore:** | 主題の正式受け入れ前に必要な単純作業。共通プロセスを参照することが多い。プロセス説明へのリンクを付ける |
| **note:** | 読者が知っておくべきことの指摘。常に non-blocking とする |

表現を広げたい場合は次の3種も使える。

- **typo:** 誤字が主題である todo
- **polish:** 内容に問題はないが、品質を即座に引き上げる余地があるという suggestion
- **quibble:** nitpick と同義。表現が好みに合う場合に使う

## 3. デコレーション

| デコレーション | 意味 |
|---|---|
| **(blocking)** | 解決されるまで主題の受け入れを妨げる |
| **(non-blocking)** | 主題の受け入れを妨げない |
| **(if-minor)** | 修正が些末で終わるなら対応する。著者に裁量を委ねる |

組織固有のデコレーション（security, test 等）も許容する。ただし1コメントに多数並べると可読性を損なうため最小限にすること。

## 4. blocking 判定規則

issue / suggestion / todo への blocking / non-blocking 付与は、影響度と発生頻度のマトリクスで決める。

### 4.1 影響度（Severity）

影響度は次の4段階とする。

- **Critical**: 設計原則違反・セキュリティ欠陥・データ破壊・回帰を伴うバグ
- **High**: 機能バグ・回帰リスク・明確な仕様不整合
- **Medium**: 保守性の低下・軽微な設計不整合・テスト不足
- **Low**: 命名・スタイル・効率に関する改善提案

### 4.2 発生頻度（Frequency）

頻度は「その問題がどの程度経験されるか」を測る。指摘の種別に応じて、判定の読み替えを使う。

**実行時に顕在化する欠陥（issue 系）は、ユーザー使用時の発生しやすさで判定する。**

- **高**: 主要な操作経路で発生する。多くのユーザーが日常的に踏む
- **中**: 特定の条件（入力、データ状態、環境）で発生する。一部のユーザーが定期的に踏む
- **低**: 稀なエッジケース。極めて限られた条件でのみ発生する

根拠として、当該コードパスへの到達条件を discussion に明記する。到達条件を静的に確定できない場合は question として著者に確認する。

**実行時に顕在化しない静的な設計問題（レイヤー違反、命名不整合、テスト不足等）は、コード内での波及範囲で判定する。**

- **高**: 同一・同種のパターンが4箇所以上。または他の開発者がコピーして増殖しつつある
- **中**: 2〜3箇所。または今後の実装で再発可能性が高い
- **低**: 単発。その場限り

波及範囲は推測で判定しない。grep または IDE の参照検索で実測し、該当箇所を discussion に列挙して根拠とする。

**両面を持つ指摘は、高い方の頻度を採る。** たとえば稀なエッジケースでしか発生しないバグのパターンが5箇所に横展開している場合、頻度は「高」とする。

### 4.3 マトリクス

| 影響度 \ 頻度 | 高 | 中 | 低 |
|---|---|---|---|
| **Critical** | blocking | blocking | blocking |
| **High** | blocking | blocking | non-blocking |
| **Medium** | blocking | non-blocking | non-blocking |
| **Low** | non-blocking | non-blocking | non-blocking |

判定原則は次の通り。

- **Critical は頻度に関わらず blocking**。影響度が最上位なら単発でも受け入れを許さない
- **頻度は High の引き下げと Medium の引き上げに効く**。稀なエッジケースでしか発生しない High は non-blocking に下がる。高頻度で発生する、または横展開した Medium は blocking に昇格する。放置コストが複利で増えるためだ
- 修正が些末なら実施する程度でよい場合は **(if-minor)** を併用する
- 対応を見送る指摘は issue にせず **thought** または **note** として記録のみ残す
- マトリクスは既定値であり、レビュアーが最終判断で上下させることを許容する。その場合は discussion に理由を明記する

## 5. コメント例

```
issue (blocking): Adapter 層を経由せず React Flow の型が Domain に流入している

Severity: High / Frequency: 高（同パターン4箇所: apps/web/src/a.ts, apps/web/src/b.ts, ...）
React Flow の型は Adapter 層でのみ使用し、Domain 型へ変換して渡すこと。
```

```
issue (non-blocking): 保存処理で非 UTF-8 の入力が渡った際に例外が発生する

Severity: High / Frequency: 低（到達条件: UI 以外の経路から非 UTF-8 の入力が供給される場合のみ。現行のフロントエンドは UTF-8 に固定）
主要経路では発生しないため non-blocking とする。ただし入力経路が増えた場合に備え、次の機会までに入力の Charset 検証を追加したい。
```

```
suggestion (if-minor): この関数の引数名が Domain の用語とずれている

命名をプロジェクトの用語集に合わせると、追跡しやすくなる。
```

## 6. 適用範囲

- レビューコメント・レビュー報告・PR/MR フィードバックでは、本スキルのフォーマットを優先して使う
- プロジェクト固有の違反レポート形式が別に定義されている場合は、本スキルはそれを置き換えない
