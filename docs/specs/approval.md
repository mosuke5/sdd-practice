# 承認・否認 — 受け入れ条件

[product-requirements.md §4.2](../product-requirements.md#42-承認者向け機能) / [api-spec.md §6.4](../api-spec.md#64-承認承認者)

**Must**: 否認時のコメントは **必須**。承認時のコメントは **任意**。

---

### APPROVAL-001: 承認者は自分宛ての申請一覧を閲覧できる

**レイヤ**: API+UI

**Given** `approver1` でログイン済みである  
**And** `approver1` 宛ての申請が複数存在する  
**When** `GET /api/approvals` を呼び出す  
**Then** レスポンスは `200 OK` である  
**And** すべての item で `approver.id` が `approver1` の ID である  
**And** 既定ソートは `createdAt` 降順である  
**And** UI の承認待ち一覧（`/approvals`）に表示される

---

### APPROVAL-002: 非承認者は承認者向け API にアクセスできない

**レイヤ**: API+UI

**Given** `applicant1`（`isApprover == false`）でログイン済みである  
**When** `GET /api/approvals` を呼び出す  
**Then** レスポンスは `403 Forbidden` である  
**And** UI では承認者向け画面（`/approvals`）にアクセスできない

---

### APPROVAL-003: 承認者は自分宛て申請の詳細を閲覧できる

**レイヤ**: API+UI

**Given** `approver1` 宛ての申請（ID = `{id}`）が存在する  
**When** `GET /api/approvals/{id}` を呼び出す  
**Then** レスポンスは `200 OK` である  
**And** 申請内容・申請者情報・`approvalActions` が含まれる

---

### APPROVAL-004: PENDING の申請を承認できる（コメント任意）

**レイヤ**: API+UI

**Given** `approver1` 宛ての申請（ID = `{id}`）が `PENDING` である  
**When** `POST /api/approvals/{id}/approve` に `{ "comment": "承認します" }` を送信する  
**Then** レスポンスは `200 OK` である  
**And** `status` は `APPROVED` である  
**And** `approvalActions` に `APPROVE` が追加される  
**And** `actedBy` は `approver1` である

---

### APPROVAL-005: コメントなしでも承認できる

**レイヤ**: API+UI

**Given** `approver1` 宛ての `PENDING` 申請が存在する  
**When** `POST /api/approvals/{id}/approve` に `{}` または `comment` 省略で送信する  
**Then** レスポンスは `200 OK` である  
**And** `APPROVE` の `comment` は `null` でもよい

---

### APPROVAL-006: PENDING の申請を否認できる（コメント必須）

**レイヤ**: API+UI

**Given** `approver1` 宛ての申請（ID = `{id}`）が `PENDING` である  
**When** `POST /api/approvals/{id}/reject` に `{ "comment": "理由を記載" }` を送信する  
**Then** レスポンスは `200 OK` である  
**And** `status` は `REJECTED` である  
**And** `approvalActions` に `REJECT` とコメントが記録される

---

### APPROVAL-007: 否認時にコメント未入力はエラー

**レイヤ**: API+UI

**Given** `approver1` 宛ての `PENDING` 申請が存在する  
**When** `POST /api/approvals/{id}/reject` に `comment` を省略、空文字、または空白のみで送信する  
**Then** レスポンスは `400 Bad Request`（`validation-error`）である  
**And** UI でも否認ボタン押下時にコメント未入力では送信できず、エラーを表示する

---

### APPROVAL-008: PENDING 以外の申請は承認・否認できない

**レイヤ**: API

**Given** 申請（ID = `{id}`）が `REJECTED` または `APPROVED` である  
**When** `POST /api/approvals/{id}/approve` または `reject` を呼び出す  
**Then** レスポンスは `409 Conflict` である  
**And** `detail` に承認・否認の対象外である旨が含まれる

---

### APPROVAL-009: 自分宛てでない申請は承認・否認できない

**レイヤ**: API

**Given** `approver1` 宛てではない申請（ID = `{id}`）が `PENDING` である  
**When** `approver1` が `POST /api/approvals/{id}/approve` を呼び出す  
**Then** レスポンスは `403 Forbidden` である

---

### APPROVAL-010: 承認者は自分宛て申請の添付をダウンロードできる

**レイヤ**: API

**Given** `approver1` 宛ての添付付き物品購入申請（ID = `{id}`）が存在する  
**When** `GET /api/approvals/{id}/attachment` を呼び出す  
**Then** レスポンスは `200 OK` であり、ファイル内容が返る

---

### APPROVAL-011: 兼務ユーザーは申請者・承認者の両機能を使える

**レイヤ**: API+UI

**Given** `dual1`（`isApprover == true`）でログイン済みである  
**When** 自分が申請者として申請を作成し、自分を承認者に指定する  
**Then** 申請作成は成功する  
**And** `GET /api/approvals` に当該申請が表示される  
**And** 承認・否認操作が可能である
