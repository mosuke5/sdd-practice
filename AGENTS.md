# AGENTS.md

申請ワークフローシステム（workflow）の **仕様・設計リポジトリ**。実装コードは別リポジトリ（`workflow-backend` / `workflow-frontend` / `workflow-manifests`）で管理する。

**正本**: [product-requirements.md](docs/product-requirements.md) が What、[design.md](docs/design.md) が How。本ファイルと design.md が矛盾する場合は design.md を正とする。実装判断で detail が必要なら design.md を読むこと。

## ドキュメント

| ファイル | 内容 | 参照タイミング |
|----------|------|----------------|
| [docs/product-requirements.md](docs/product-requirements.md) | プロダクト要求（What） | 機能・ワークフロー・スコープの確認 |
| [docs/design.md](docs/design.md) | 技術設計（How） | アーキテクチャ・技術選定・実装方針 |
| `docs/api-spec.md` | API 仕様（正式定義） | エンドポイント・ワークフロー・副作用の確認 |
| [docs/specs/](docs/specs/) | 機能別受け入れ条件 | Given / When / Then 形式 |

本ファイルはエージェント向けの **Must 制約** と行動規範のみを記載する。システム概要・リポジトリ構成・技術スタック・ローカル開発手順等は [design.md](docs/design.md) を参照。

## Must 制約（実装時に例外なく守る）

### ドメイン

- **種別**: `LEAVE` / `PURCHASE` — 詳細: [design.md §4.3](docs/design.md#43-主要ドメインモデル案)
- **ステータス**: `PENDING` / `APPROVED` / `REJECTED`
- **再申請**: 否認後は **同一 `Application` レコードを更新**する（新規レコードは作らない）。申請 ID・URL は変わらない
- **`APPROVED` の申請**は再申請・再編集不可
- 否認・承認の履歴は `ApprovalAction` に蓄積する
- **否認時のコメントは必須**（未入力は API・UI ともエラー）。承認時のコメントは任意 — 詳細: [design.md §4.4](docs/design.md#44-承認ワークフロー)

### 認可

- 申請者: **自分の申請のみ**操作可能
- 承認者: **自分宛ての申請のみ**操作可能
- 承認者向け画面は承認者フラグ（`is_approver`）が有効なユーザーのみアクセス可能 — 詳細: [design.md §4.6](docs/design.md#46-横断関心事)

### 添付ファイル（物品購入申請のみ・任意）

- 最大 **3 MB**（3,145,728 バイト）
- 許可形式: `.pdf` / `.xls` / `.xlsx`
- 拡張子と MIME タイプの **両方** で検証する
- ファイル本体は S3 互換オブジェクトストレージ、DB はメタデータのみ — 詳細: [design.md §4.5](docs/design.md#45-添付ファイル)

### 通知

| イベント | 通知先 |
|----------|--------|
| 申請送信 | 承認者 |
| 承認・否認 | 申請者 |
| 再申請 | 承認者 |

— 詳細: [design.md §7](docs/design.md#7-通知設計)

### 技術（Must のみ）

- API エラーは **RFC 7807** Problem Details（`application/problem+json`）。フロントは `detail` をユーザー向けメッセージとして表示 — 詳細: [design.md §3.5](docs/design.md#35-エラーレスポンスrfc-7807)
- 認証は **セッション（Cookie）**。パスワードは **BCrypt** でハッシュ化。セッション保存は **インメモリ** — 詳細: [design.md §6](docs/design.md#6-認証セキュリティ)
- API の正式定義は **[api-spec.md](docs/api-spec.md)**。OpenAPI / `openapi.yaml` は採用しない（[design.md D-17](docs/design.md#101-決定済み)）

## スコープ外（初版 — 実装しない）

- OIDC 連携（初版は本システム内認証）
- 多段承認
- 申請の取り下げ・取消
- ユーザー管理画面・管理 API（初期ユーザーは Flyway シードで投入）
- メール以外の通知（Slack 等）
- 監査ログ・レポート

— 詳細: [product-requirements.md §8](docs/product-requirements.md) / [design.md §4.2.1](docs/design.md#421-ユーザー管理初版)

## 未決定事項（TBD — 独断で決めない）

[design.md §10.2](docs/design.md#102-未決定) および各所の TBD を確認し、仮実装せずドキュメントに「未決定」と残すか、ユーザーに確認する。

## エージェント向けガイドライン

### このリポジトリで行うこと

- `docs/` の要求・設計・API ドキュメントの作成・更新
- `api-spec.md` / `docs/specs/` / 受け入れ条件の整備
- 設計上の未決定事項の明文化

### このリポジトリで行わないこと

- アプリケーションコードの実装（各 `workflow-*` リポジトリで行う）
- プラットフォーム CI/CD パイプライン定義の変更

### 実装・仕様変更時の原則

1. **product-requirements.md が What、design.md が How の正** — 矛盾する場合はドキュメントを先に更新する
2. 上記 Must 制約は **例外なく** 守る
3. API 変更は [api-spec.md](docs/api-spec.md) を先に更新し、実装と同期する
4. スコープ外機能を勝手に追加しない
5. TBD 項目は仮実装せず、ドキュメントに「未決定」と残すか、ユーザーに確認する

## 参照リンク（詳細）

| トピック | 参照先 |
|----------|--------|
| システム概要・ロール | [product-requirements.md](docs/product-requirements.md) |
| アーキテクチャ・リポジトリ構成 | [design.md §2](docs/design.md#2-システム構成) |
| 技術スタック | [design.md §3](docs/design.md#3-技術スタック) |
| バックエンド設計 | [design.md §4](docs/design.md#4-バックエンド設計) |
| フロントエンド設計・画面一覧 | [design.md §5](docs/design.md#5-フロントエンド設計) |
| 認証・CORS | [design.md §6](docs/design.md#6-認証セキュリティ) |
| ローカル開発（Podman 等） | [design.md §9](docs/design.md#9-ローカル開発環境方針) |
| API エンドポイント・認可・ワークフロー | [docs/api-spec.md](docs/api-spec.md) |
| 決定済み・未決定事項 | [design.md §10](docs/design.md#10-決定事項未決定事項) |
| 次のステップ | [design.md §11](docs/design.md#11-次のステップ) |
