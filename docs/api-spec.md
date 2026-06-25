# API 仕様書

## 1. 概要

### 1.1 目的

本ドキュメントは、申請ワークフローシステムの REST API について、エンドポイント・認可・ビジネスルール・副作用の **正式定義** とする。実装（`workflow-backend` / `workflow-frontend`）は本書に整合させる。

### 1.2 関連ドキュメント

| ドキュメント | 説明 |
|--------------|------|
| [product-requirements.md](./product-requirements.md) | プロダクト要求（What） |
| [design.md](./design.md) | 技術設計（How） |
| api-spec.md（本書） | API の正式定義 |
| [specs/](./specs/) | 機能別受け入れ条件 |

---

## 2. 共通仕様

### 2.1 ベース URL

| 環境 | ベース URL |
|------|------------|
| ローカル開発 | `http://localhost:8080` |
| 検証・本番 | **TBD**（OpenShift Route 確定後） |

- すべてのエンドポイントは `/api` プレフィックスを付ける（例: `POST /api/auth/login`）。
- フロントエンドは環境変数でベース URL を切り替える（変数名は **TBD**。Nuxt では `NUXT_PUBLIC_API_BASE_URL` を想定）。

### 2.2 認証

| 項目 | 仕様 |
|------|------|
| 方式 | セッション（Cookie） |
| クライアント | `fetch` 等で `credentials: 'include'` を指定 |
| 未認証 | `401 Unauthorized`（ログイン API を除く） |
| CORS（ローカル） | 許可オリジン `http://localhost:3000`、`Allow-Credentials: true` |

### 2.3 リクエスト・レスポンス形式

| 種別 | Content-Type |
|------|--------------|
| JSON API | `application/json` |
| ファイルアップロード | `multipart/form-data` |
| エラー | `application/problem+json`（RFC 7807） |

- 日時は ISO 8601（UTC またはオフセット付き）文字列とする（例: `2026-06-25T10:00:00+09:00`）。
- 日付のみの項目（休暇 from/to）は `YYYY-MM-DD` 形式とする。

### 2.4 エラーレスポンス

[design.md §3.5](./design.md#35-エラーレスポンスrfc-7807) に準拠。`ProblemDetail` スキーマは本書 §2.4 で定義する。

**標準フィールド**: `type`, `title`, `status`, `detail`, `instance`

**バリデーションエラー（複数フィールド）**

`type` が `validation-error` のとき、拡張フィールド `errors` を付与する。

```json
{
  "type": "https://workflow.example.com/problems/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "入力内容に誤りがあります",
  "instance": "/api/applications",
  "errors": [
    { "field": "leaveReason", "message": "休暇理由は必須です" },
    { "field": "comment", "message": "否認時はコメントが必須です" }
  ]
}
```

**問題 type URI（例）**

| type 末尾 | HTTP | 用途 |
|-----------|------|------|
| `validation-error` | 400 | 入力バリデーション |
| `unauthorized` | 401 | 未認証 |
| `forbidden` | 403 | 認可エラー |
| `not-found` | 404 | リソース未存在 |
| `conflict` | 409 | 状態遷移の競合 |
| `internal-error` | 500 | サーバー内部エラー |

### 2.5 ページネーション

初版の一覧 API は **ページネーションなし** とする。件数増加時に `page` / `size` クエリを追加する（将来対応）。

- 一覧の既定ソート: `createdAt` 降順（新しい順）。

---

## 3. 列挙型・共通モデル

### 3.1 列挙型

| 名前 | 値 | 説明 |
|------|-----|------|
| ApplicationType | `LEAVE` | 休暇申請 |
| ApplicationType | `PURCHASE` | 物品購入申請 |
| ApplicationStatus | `PENDING` | 申請中（承認待ち） |
| ApplicationStatus | `APPROVED` | 承認済み |
| ApplicationStatus | `REJECTED` | 否認 |
| ApprovalActionType | `APPROVE` | 承認 |
| ApprovalActionType | `REJECT` | 否認 |
| ApprovalActionType | `RESUBMIT` | 再申請（否認後の再提出） |

### 3.2 UserSummary

一覧・参照用のユーザー情報。

| フィールド | 型 | 説明 |
|------------|-----|------|
| id | integer | ユーザー ID |
| loginId | string | ログイン ID |
| displayName | string | 表示名 |
| isApprover | boolean | 承認者フラグ |

### 3.3 UserMe

`GET /api/auth/me` 用。`UserSummary` に加え、ログイン中ユーザー向け情報。

| フィールド | 型 | 説明 |
|------------|-----|------|
| id | integer | ユーザー ID |
| loginId | string | ログイン ID |
| displayName | string | 表示名 |
| email | string | メールアドレス（通知送信先） |
| isApprover | boolean | 承認者フラグ |

### 3.4 LeaveDetail

| フィールド | 型 | 必須 | 説明 |
|------------|-----|------|------|
| leaveFrom | string (date) | ✓ | 休暇開始日 |
| leaveTo | string (date) | ✓ | 休暇終了日 |
| leaveReason | string | ✓ | 休暇理由 |

- `leaveFrom` は `leaveTo` 以前であること。

### 3.5 PurchaseDetail

| フィールド | 型 | 必須 | 説明 |
|------------|-----|------|------|
| itemName | string | ✓ | 品名 |
| amount | integer | ✓ | 金額（円、0 以上） |

### 3.6 AttachmentInfo

| フィールド | 型 | 説明 |
|------------|-----|------|
| id | integer | 添付 ID |
| fileName | string | 元ファイル名 |
| contentType | string | MIME タイプ |
| sizeBytes | integer | サイズ（バイト） |

- 物品購入申請に **0 または 1 件** の添付を許可する。

### 3.7 ApprovalActionInfo

| フィールド | 型 | 説明 |
|------------|-----|------|
| id | integer | アクション ID |
| actionType | ApprovalActionType | 種別 |
| comment | string \| null | コメント |
| actedBy | UserSummary | 実行者 |
| actedAt | string (datetime) | 実行日時 |

### 3.8 ApplicationSummary

一覧用。

| フィールド | 型 | 説明 |
|------------|-----|------|
| id | integer | 申請 ID |
| type | ApplicationType | 申請種別 |
| status | ApplicationStatus | ステータス |
| applicant | UserSummary | 申請者 |
| approver | UserSummary | 承認者 |
| createdAt | string (datetime) | 申請日時 |
| updatedAt | string (datetime) | 最終更新日時 |
| title | string | 一覧表示用の要約（種別に応じて生成。例: 休暇 2026-06-01〜06-03） |

### 3.9 ApplicationDetail

詳細用。`ApplicationSummary` のフィールドに加え:

| フィールド | 型 | 説明 |
|------------|-----|------|
| leaveDetail | LeaveDetail \| null | 休暇申請時のみ |
| purchaseDetail | PurchaseDetail \| null | 物品購入申請時のみ |
| attachment | AttachmentInfo \| null | 物品購入・添付あり時のみ |
| approvalActions | ApprovalActionInfo[] | 承認・否認・再申請の履歴（`actedAt` 昇順） |

---

## 4. 認可ルール

### 4.1 原則

| ロール | 条件 | 操作範囲 |
|--------|------|----------|
| 認証済みユーザー | ログイン済み | 自分自身の情報、申請の作成 |
| 申請者 | `application.applicantId == 自分` | 自分の申請の参照・再申請 |
| 承認者 | `user.isApprover == true` | 承認者向け API |
| 承認者（対象申請） | `application.approverId == 自分` | 自分宛て申請の参照・承認・否認 |

### 4.2 エンドポイント別認可マトリクス

| エンドポイント | 未認証 | 申請者 | 承認者 | 備考 |
|----------------|--------|--------|--------|------|
| `POST /api/auth/login` | ✓ | — | — | |
| `POST /api/auth/logout` | | ✓ | ✓ | |
| `GET /api/auth/me` | | ✓ | ✓ | |
| `GET /api/users/approvers` | | ✓ | ✓ | 申請作成時の承認者選択用 |
| `GET /api/applications` | | ✓（自分のみ） | | |
| `POST /api/applications` | | ✓ | ✓ | 全認証ユーザーが申請可能 |
| `GET /api/applications/{id}` | | ✓（自分のみ） | | |
| `PUT /api/applications/{id}` | | ✓（自分・REJECTED のみ） | | 再申請 |
| `GET /api/applications/{id}/attachment` | | ✓（自分のみ） | | 申請者は自分の申請 |
| `GET /api/approvals` | | | ✓ | `isApprover` 必須 |
| `GET /api/approvals/{id}` | | | ✓（自分宛てのみ） | |
| `GET /api/approvals/{id}/attachment` | | | ✓（自分宛てのみ） | |
| `POST /api/approvals/{id}/approve` | | | ✓（自分宛て・PENDING のみ） | |
| `POST /api/approvals/{id}/reject` | | | ✓（自分宛て・PENDING のみ） | |

- 承認者も申請者として自分の申請を作成・閲覧できる。
- 他人の申請へのアクセスは `403 Forbidden`。

---

## 5. ワークフロー・状態遷移

### 5.1 状態遷移

```
[新規申請] ──POST /api/applications──► PENDING
PENDING ──approve──► APPROVED（終了）
PENDING ──reject──► REJECTED
REJECTED ──PUT /api/applications/{id}（再申請）──► PENDING
```

### 5.2 操作ごとの前提条件

| 操作 | 許可ステータス | その他 |
|------|----------------|--------|
| 申請作成 | — | 新規 `Application` を `PENDING` で作成 |
| 再申請（更新） | `REJECTED` のみ | 同一 `Application` を更新。`APPROVED` / `PENDING` は不可 |
| 承認 | `PENDING` のみ | 自分宛て、承認者のみ |
| 否認 | `PENDING` のみ | 自分宛て、承認者のみ。コメント必須 |

### 5.3 再申請時の履歴

- 再申請（`PUT /api/applications/{id}`）成功時、`ApprovalAction` に `RESUBMIT` を 1 件記録する（コメントは任意、`null` 可）。
- 過去の `APPROVE` / `REJECT` 履歴は削除しない。

### 5.4 競合時のエラー

| 状況 | HTTP | detail 例 |
|------|------|-----------|
| `APPROVED` への再申請 | 409 | 承認済みの申請は更新できません |
| `PENDING` への再申請 | 409 | 審査中の申請は更新できません |
| `PENDING` 以外への承認・否認 | 409 | この申請は承認・否認の対象ではありません |

---

## 6. エンドポイント詳細

### 6.1 認証

#### `POST /api/auth/login`

ログインし、セッション Cookie を発行する。

**認証**: 不要

**リクエスト**

```json
{
  "loginId": "applicant1",
  "password": "password"
}
```

| フィールド | 型 | 必須 |
|------------|-----|------|
| loginId | string | ✓ |
| password | string | ✓ |

**レスポンス**: `200 OK`

```json
{
  "user": {
    "id": 1,
    "loginId": "applicant1",
    "displayName": "申請 太郎",
    "email": "applicant1@example.com",
    "isApprover": false
  }
}
```

- レスポンスヘッダに `Set-Cookie`（セッション）を含む。

**エラー**

| 状況 | HTTP |
|------|------|
| ID・パスワード不一致 | 401 |

---

#### `POST /api/auth/logout`

セッションを無効化する。

**認証**: 要

**レスポンス**: `204 No Content`

---

#### `GET /api/auth/me`

ログイン中のユーザー情報を返す。

**認証**: 要

**レスポンス**: `200 OK` — `UserMe`

**エラー**

| 状況 | HTTP |
|------|------|
| 未ログイン | 401 |

---

### 6.2 ユーザー

#### `GET /api/users/approvers`

承認者候補一覧を返す。`isApprover == true` のユーザーのみ。

**認証**: 要

**レスポンス**: `200 OK`

```json
{
  "items": [
    {
      "id": 2,
      "loginId": "approver1",
      "displayName": "承認 花子",
      "isApprover": true
    }
  ]
}
```

- ソート: `displayName` 昇順。

---

### 6.3 申請（申請者）

#### `GET /api/applications`

自分が申請した申請の一覧。

**認証**: 要

**クエリ**（任意）

| パラメータ | 型 | 説明 |
|------------|-----|------|
| status | ApplicationStatus | ステータスでフィルタ |

**レスポンス**: `200 OK`

```json
{
  "items": [ /* ApplicationSummary[] */ ]
}
```

---

#### `POST /api/applications`

申請を新規作成し、`PENDING` で登録する。作成後、承認者へメール通知する。

**認証**: 要

**リクエスト（休暇・JSON）**

`Content-Type: application/json`

```json
{
  "type": "LEAVE",
  "approverId": 2,
  "leaveDetail": {
    "leaveFrom": "2026-07-01",
    "leaveTo": "2026-07-03",
    "leaveReason": "私用のため"
  }
}
```

**リクエスト（物品購入・multipart）**

`Content-Type: multipart/form-data`

| パート | 名前 | 型 | 必須 | 説明 |
|--------|------|-----|------|------|
| メタデータ | `data` | JSON 文字列 | ✓ | 下記 `PurchaseCreateData` |
| ファイル | `file` | binary | | 添付（任意） |

`PurchaseCreateData` 例:

```json
{
  "type": "PURCHASE",
  "approverId": 2,
  "purchaseDetail": {
    "itemName": "ノート PC スタンド",
    "amount": 5000
  }
}
```

**共通バリデーション**

| ルール | エラー |
|--------|--------|
| `approverId` は存在し、`isApprover == true` | 400 |
| `type` に応じた detail が必須 | 400 |
| `LEAVE` に `purchaseDetail` / ファイルあり | 400 |
| `PURCHASE` に `leaveDetail` あり | 400 |

**レスポンス**: `201 Created` — `ApplicationDetail`

**副作用**: 承認者へ新規申請通知メール（§8）。

---

#### `GET /api/applications/{id}`

自分の申請の詳細。

**認証**: 要（申請者本人）

**レスポンス**: `200 OK` — `ApplicationDetail`

**エラー**

| 状況 | HTTP |
|------|------|
| 存在しない ID | 404 |
| 他人の申請 | 403 |

---

#### `PUT /api/applications/{id}`

否認された申請を修正して再申請する。ステータスを `PENDING` に戻す。

**認証**: 要（申請者本人）

**前提**: `status == REJECTED`

**リクエスト**: `POST /api/applications` と同様（種別は変更不可。作成時の `type` と一致すること）

- 休暇: `application/json`
- 物品購入: `multipart/form-data`（`file` パートで添付の差し替え可。省略時は既存添付を維持）

**レスポンス**: `200 OK` — `ApplicationDetail`

**副作用**

- `ApprovalAction` に `RESUBMIT` を記録
- 承認者へ再申請通知メール（§8）

---

#### `GET /api/applications/{id}/attachment`

申請の添付ファイルをダウンロードする。

**認証**: 要（申請者本人）

**前提**: 物品購入申請かつ添付が存在

**レスポンス**: `200 OK`

- `Content-Type`: 保存時の MIME タイプ
- `Content-Disposition: attachment; filename="..."`

**エラー**

| 状況 | HTTP |
|------|------|
| 添付なし | 404 |

---

### 6.4 承認（承認者）

#### `GET /api/approvals`

自分宛ての申請一覧。

**認証**: 要（`isApprover == true`）

**クエリ**（任意）

| パラメータ | 型 | 説明 |
|------------|-----|------|
| status | ApplicationStatus | ステータスでフィルタ（未指定時は全件） |

**レスポンス**: `200 OK`

```json
{
  "items": [ /* ApplicationSummary[] */ ]
}
```

- `approverId == 自分` の申請のみ。

**エラー**

| 状況 | HTTP |
|------|------|
| 非承認者 | 403 |

---

#### `GET /api/approvals/{id}`

自分宛て申請の詳細。

**認証**: 要（承認者・自分宛て）

**レスポンス**: `200 OK` — `ApplicationDetail`

---

#### `GET /api/approvals/{id}/attachment`

自分宛て申請の添付ファイルをダウンロード。

**認証**: 要（承認者・自分宛て）

**レスポンス**: `GET /api/applications/{id}/attachment` と同様。

---

#### `POST /api/approvals/{id}/approve`

申請を承認する。

**認証**: 要（承認者・自分宛て）

**前提**: `status == PENDING`

**リクエスト**

```json
{
  "comment": "承認します"
}
```

| フィールド | 型 | 必須 |
|------------|-----|------|
| comment | string | | 任意 |

**レスポンス**: `200 OK` — `ApplicationDetail`（`status: APPROVED`）

**副作用**

- `ApprovalAction`（`APPROVE`）を記録
- 申請者へ承認通知メール（§8）

---

#### `POST /api/approvals/{id}/reject`

申請を否認する。

**認証**: 要（承認者・自分宛て）

**前提**: `status == PENDING`

**リクエスト**

```json
{
  "comment": "金額の根拠資料を添付してください"
}
```

| フィールド | 型 | 必須 |
|------------|-----|------|
| comment | string | **✓** |

- 空文字・空白のみは不可。

**レスポンス**: `200 OK` — `ApplicationDetail`（`status: REJECTED`）

**副作用**

- `ApprovalAction`（`REJECT`）を記録
- 申請者へ否認通知メール（コメント含む）（§8）

---

## 7. 添付ファイル

### 7.1 制約

[design.md §4.5](./design.md#45-添付ファイル) / [product-requirements.md §3.3](./product-requirements.md#33-物品購入申請)

| 項目 | 値 |
|------|-----|
| 対象 | 物品購入申請（`PURCHASE`）のみ |
| 件数 | 0 または 1 件 |
| 最大サイズ | 3 MB（3,145,728 バイト） |
| 許可拡張子 | `.pdf`, `.xls`, `.xlsx` |
| 許可 MIME | `application/pdf`, `application/vnd.ms-excel`, `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` |

- 拡張子と MIME の **両方** を検証する。
- 違反時は `400`（`validation-error`）。

### 7.2 オブジェクトストレージ

| 項目 | 値 |
|------|-----|
| API | S3 互換 |
| バケット（ローカル dev） | `workflow-attachments-dev` |
| バケット（検証・本番） | `workflow-attachments`（環境サフィックスは manifests で上書き可） |

**オブジェクトキー形式**

```
applications/{applicationId}/attachment/{attachmentId}/{sanitizedFileName}
```

- `sanitizedFileName` はパス区切り・制御文字を除去した元ファイル名。

### 7.3 再申請時の添付

- `file` パートあり: 既存オブジェクトを削除し新規保存（DB メタデータも更新）。
- `file` パートなし: 既存添付を維持。
- 既存添付の明示的削除のみ（添付なしに変更）は初版では **不可**（常に 0 または 1 件。削除 API は設けない）。

---

## 8. 通知（メール副作用）

API 成功時に非同期または同期で SMTP 送信する。失敗時の扱い（リトライ・ログのみ）は実装時に決定するが、**API 自体の成否とは切り離し**、申請データのコミットを優先する（メール失敗で申請作成をロールバックしない）。

### 8.1 通知トリガー

| API | 通知先 | 件名（例） |
|-----|--------|------------|
| `POST /api/applications` | 承認者 | `[Workflow] 新規申請が届きました` |
| `PUT /api/applications/{id}`（再申請） | 承認者 | `[Workflow] 申請が再提出されました` |
| `POST /api/approvals/{id}/approve` | 申請者 | `[Workflow] 申請が承認されました` |
| `POST /api/approvals/{id}/reject` | 申請者 | `[Workflow] 申請が否認されました` |

### 8.2 メール本文に含める情報

| 項目 | 内容 |
|------|------|
| 申請 ID | `{id}` |
| 申請種別 | 休暇 / 物品購入 |
| 要約 | `ApplicationSummary.title` 相当 |
| 申請者 / 承認者名 | 通知先に応じて |
| コメント | 承認・否認時（否認は必須のため常に含む） |
| 詳細リンク | フロントエンド URL（下記） |

**詳細リンク**

| 通知先 | URL 形式 |
|--------|----------|
| 承認者 | `{frontendBaseUrl}/approvals/{id}` |
| 申請者 | `{frontendBaseUrl}/applications/{id}` |

- `frontendBaseUrl` は環境ごとの設定（ローカル: `http://localhost:3000`、検証・本番は **TBD**）。

---

## 9. 仕様変更時の原則

- API を変更する場合は、**本書（api-spec.md）を先に更新**し、続けて `workflow-backend` / `workflow-frontend` の実装を変更する。
- [product-requirements.md](./product-requirements.md) / [design.md](./design.md) と矛盾する変更は行わない。
- 受け入れ条件（`docs/specs/`）がある場合は、あわせて更新する。

---

## 10. スコープ外（API として提供しない）

[product-requirements.md §8](./product-requirements.md#8-スコープ外初版)

- ユーザー CRUD（管理 API）
- 申請の取り下げ・取消
- OIDC ログイン
- 多段承認
- Webhook / Slack 通知

---

## 11. 改訂履歴

| 版 | 日付 | 内容 |
|----|------|------|
| 0.1 | 2026-06-25 | 初版作成 |
| 0.2 | 2026-06-25 | OpenAPI / `openapi.yaml` 非採用。本書を API 正式定義とする |
