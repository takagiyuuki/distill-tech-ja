# distill-tech-ja — Claude Code 運用ルール

> 作業開始時に正典を読むこと: ドメイン用語は `CONTEXT.md`、覆しにくい設計判断は `docs/adr/`、該当する GitHub issue。これらと本ファイルが食い違ったら正典側を優先。

## モード宣言（実装委任モード）

このリポジトリは **実装委任モード**（グローバル `CLAUDE.md` の Implementation Delegation Mode）で運用する。学習用リポジトリのデフォルト（明示指示なしの Edit/Write 禁止・事後報告禁止）は適用しない。

- 破壊的・非可逆な操作、および外向きの操作（push / PR 作成 / `wrangler deploy` / `terraform apply` / シークレット操作 / 削除）は事前に確認する。
- 委任モードでも「結論から簡潔に」「事実と推測の区別」「設計懸念の指摘」は守る。

## 役割分担

目的は「転職活動で設計・運用判断を自分の言葉で説明できること」。実装の手を動かすのは主に Claude、判断と最終責任は私が持つ。

| 担当 | 範囲 |
| --- | --- |
| 私 | 設計判断（ADR）、issue での仕様確定、全 PR のレビューとマージ、SLO・監視設計 |
| Claude | アプリ層の実装（Hono / D1 アクセス / 要約パイプライン / アダプタ）、テスト、Terraform・CI の下書き |
| 共同 | Terraform・GitHub Actions（Claude が下書き、私が1行ずつ理解してからマージ） |

- 新しい設計判断が必要になったら、実装で既成事実化せず止めて相談する。合意したら ADR 案を書く。
- PR 説明には「何を・なぜ・検討した代替案・レビューで見てほしい点」を書く。私が面接で説明できる粒度にする。

## ワークフロー

- 1 issue = 1 ブランチ = 1 PR。`main` へ直接コミットしない。
- CI（lint / typecheck / test）が通らない PR は出さない。
- **マージは私が行う。** Claude はマージしない。
- VCS は Git を使う（`.jj/` があっても `jj` は使わない）。コミットメッセージは英語・Conventional Commits。

## 技術スタック

- Hono + Cloudflare Workers / Workers AI / D1 / Cron Triggers
- 推論は Workers AI のみ（外部LLM APIは使わない）
- IaC: Terraform（アカウントレベルのリソース）+ wrangler（Worker 本体）。境界は ADR で確定する
- CI/CD: GitHub Actions

## 設計上の不変条件

- 記事本文・コメント原文をDBや配信に含めない。要約と原典URLのみ。
- 取得対象の robots.txt / 利用規約を尊重する。
- 取得層はソース別アダプタとして抽象化する（HN専用API / RSS の差を吸収）。
- シークレット（Webhook URL・API トークン等）はコードやリポジトリに書かない。

## 進め方

- 3ファイル以上に触れる変更・スキーマ変更は、必ずプランモードで計画を先に出す。
- 計画と実態がズレたら、勝手に進めず一度止めて相談する。
