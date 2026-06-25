# 認可 — 受け入れ条件

[api-spec.md §4](../api-spec.md#4-認可ルール) / [design.md §4.6](../design.md#46-横断関心事)

---

### AUTHZ-001: 申請者は自分の申請のみ参照できる

**レイヤ**: API

**Given** `applicant1` が作成した申請（ID = `{id}`）が存在する  
**And** `dual1` でログイン済みである（`dual1` は当該申請の申請者ではない）  
**When** `GET /api/applications/{id}` を呼び出す  
**Then** レスポンスは `403 Forbidden` である

---

### AUTHZ-002: 申請者は他人の申請を一覧に含めない

**レイヤ**: API

**Given** `applicant1` と `dual1` がそれぞれ申請を作成している  
**When** `applicant1` が `GET /api/applications` を呼び出す  
**Then** 返却される `items` に `dual1` の申請は含まれない

---

### AUTHZ-003: 承認者は自分宛てでない申請詳細を参照できない

**レイヤ**: API

**Given** `approver1` 宛てではない申請（ID = `{id}`）が存在する  
**When** `approver1` が `GET /api/approvals/{id}` を呼び出す  
**Then** レスポンスは `403 Forbidden` である

---

### AUTHZ-004: 承認者向け API は isApprover 必須

**レイヤ**: API+UI

**Given** `applicant1`（`isApprover == false`）でログイン済みである  
**When** 次のいずれかを呼び出す  
- `GET /api/approvals`  
- `GET /api/approvals/{id}`  
- `POST /api/approvals/{id}/approve`  
- `POST /api/approvals/{id}/reject`  
**Then** レスポンスは `403 Forbidden` である

---

### AUTHZ-005: 存在しない申請 ID は 404

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**When** 存在しない ID で `GET /api/applications/999999` を呼び出す  
**Then** レスポンスは `404 Not Found` である

---

### AUTHZ-006: 全認証ユーザーは申請を作成できる

**レイヤ**: API

**Given** `approver1`（承認者）でログイン済みである  
**When** 有効な申請内容で `POST /api/applications` を呼び出す  
**Then** レスポンスは `201 Created` である  
**And** 承認者も申請者として申請作成可能である

---

### AUTHZ-007: 未認証は保護エンドポイントに 401

**レイヤ**: API

**Given** 未ログイン状態である  
**When** 次のいずれかを Cookie なしで呼び出す  
- `GET /api/applications`  
- `POST /api/applications`  
- `GET /api/approvals`  
- `POST /api/auth/logout`  
**Then** レスポンスは `401 Unauthorized` である  
**And** `POST /api/auth/login` のみ未認証アクセス可

---

### AUTHZ-008: エラーレスポンスは RFC 7807 形式

**レイヤ**: API+UI

**Given** 認可エラーまたはバリデーションエラーが発生する操作を実行する  
**When** エラーレスポンスを受け取る  
**Then** `Content-Type` は `application/problem+json` である  
**And** `type`, `title`, `status`, `detail` が含まれる  
**And** UI では `detail` をユーザー向けメッセージとして表示する
