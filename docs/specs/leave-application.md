# 休暇申請 — 受け入れ条件

[product-requirements.md §3.2](../product-requirements.md#32-休暇申請) / [api-spec.md §6.3](../api-spec.md#63-申請申請者)

---

### LEAVE-001: 休暇申請を新規作成できる

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**And** `approver1` が承認者（`isApprover == true`）として存在する  
**When** `POST /api/applications` に次の JSON を送信する

```json
{
  "type": "LEAVE",
  "approverId": "<approver1 の ID>",
  "leaveDetail": {
    "leaveFrom": "2026-07-01",
    "leaveTo": "2026-07-03",
    "leaveReason": "私用のため"
  }
}
```

**Then** レスポンスは `201 Created` である  
**And** `ApplicationDetail.status` は `PENDING` である  
**And** `ApplicationDetail.type` は `LEAVE` である  
**And** `leaveDetail` がリクエスト内容と一致する  
**And** `approvalActions` は空配列である  
**And** UI では申請作成画面から送信後、申請詳細または一覧で `PENDING` が表示される

---

### LEAVE-002: 自分の休暇申請一覧を閲覧できる

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**And** `applicant1` が休暇申請を 2 件以上作成している  
**When** `GET /api/applications` を呼び出す  
**Then** レスポンスは `200 OK` である  
**And** `items` には `applicant1` が申請した申請のみが含まれる  
**And** 各 item に `type`, `status`, `createdAt`, `approver`, `title` が含まれる  
**And** 既定ソートは `createdAt` 降順（新しい順）である

---

### LEAVE-003: ステータスで申請一覧をフィルタできる

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**And** `PENDING` と `REJECTED` の休暇申請がそれぞれ存在する  
**When** `GET /api/applications?status=REJECTED` を呼び出す  
**Then** レスポンスの `items` はすべて `status == REJECTED` である

---

### LEAVE-004: 自分の休暇申請詳細を閲覧できる

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**And** `applicant1` が作成した休暇申請（ID = `{id}`）が存在する  
**When** `GET /api/applications/{id}` を呼び出す  
**Then** レスポンスは `200 OK` である  
**And** `leaveDetail` および `approvalActions` が含まれる  
**And** UI では申請詳細画面に休暇期間・理由・ステータス・承認履歴が表示される

---

### LEAVE-005: 必須項目未入力で申請できない

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** `leaveReason` を省略したリクエストで `POST /api/applications` を呼び出す  
**Then** レスポンスは `400 Bad Request` である  
**And** `type` が `validation-error` の Problem Details である  
**And** `errors` 配列に該当フィールドのエラーが含まれる  
**And** UI でも送信前または送信後にバリデーションエラーを表示する

---

### LEAVE-006: 休暇開始日が終了日より後で申請できない

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** `leaveFrom` が `leaveTo` より後の日付で申請する  
**Then** レスポンスは `400 Bad Request`（`validation-error`）である

---

### LEAVE-007: 非承認者を指定して申請できない

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**And** `applicant1` は `isApprover == false` である  
**When** `approverId` に `applicant1` の ID を指定して申請する  
**Then** レスポンスは `400 Bad Request` である

---

### LEAVE-008: 休暇申請に物品購入 detail やファイルを含められない

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**When** `type: LEAVE` のリクエストに `purchaseDetail` または multipart の `file` パートを含める  
**Then** レスポンスは `400 Bad Request` である

---

### LEAVE-009: 承認者候補一覧に isApprover のユーザーのみ含まれる

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** `GET /api/users/approvers` を呼び出す  
**Then** レスポンスは `200 OK` である  
**And** すべての item で `isApprover == true` である  
**And** `applicant1` は含まれない  
**And** `displayName` 昇順でソートされている  
**And** UI の承認者選択肢にも非承認者は表示されない
