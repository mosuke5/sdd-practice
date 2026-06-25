# 物品購入申請 — 受け入れ条件

[product-requirements.md §3.3](../product-requirements.md#33-物品購入申請) / [api-spec.md §6.3, §7](../api-spec.md#63-申請申請者)

---

### PURCHASE-001: 添付なしで物品購入申請を作成できる

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**And** `approver1` が存在する  
**When** `POST /api/applications` を `multipart/form-data` で送信する  
**And** `data` パートに次の JSON を含める

```json
{
  "type": "PURCHASE",
  "approverId": "<approver1 の ID>",
  "purchaseDetail": {
    "itemName": "ノート PC スタンド",
    "amount": 5000
  }
}
```

**And** `file` パートは含めない  
**Then** レスポンスは `201 Created` である  
**And** `status` は `PENDING`、`type` は `PURCHASE` である  
**And** `attachment` は `null` である

---

### PURCHASE-002: PDF 添付付きで物品購入申請を作成できる

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** 有効な PDF ファイル（拡張子 `.pdf`、MIME `application/pdf`、3 MB 以下）を `file` パートに添付して申請する  
**Then** レスポンスは `201 Created` である  
**And** `attachment` に `fileName`, `contentType`, `sizeBytes` が含まれる  
**And** オブジェクトストレージにファイル本体が保存される  
**And** DB にはメタデータのみが保存される

---

### PURCHASE-003: Excel 添付（.xlsx / .xls）で申請できる

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**When** 許可された Excel ファイル（`.xlsx` + 対応 MIME、または `.xls` + 対応 MIME）を添付して申請する  
**Then** レスポンスは `201 Created` である

---

### PURCHASE-004: 3 MB を超えるファイルは拒否される

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** 3,145,729 バイト以上のファイルを添付して申請する  
**Then** レスポンスは `400 Bad Request`（`validation-error`）である  
**And** UI でもエラーメッセージ（`detail`）を表示する

---

### PURCHASE-005: 許可外拡張子のファイルは拒否される

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** 拡張子が `.docx` 等、許可外のファイルを添付して申請する  
**Then** レスポンスは `400 Bad Request`（`validation-error`）である

---

### PURCHASE-006: 拡張子と MIME の不一致は拒否される

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**When** 拡張子は `.pdf` だが Content-Type が `image/png` 等、不一致のファイルを送信する  
**Then** レスポンスは `400 Bad Request`（`validation-error`）である

---

### PURCHASE-007: 申請者は自分の申請の添付をダウンロードできる

**レイヤ**: API+UI

**Given** `applicant1` が添付付き物品購入申請（ID = `{id}`）を作成済みである  
**When** `GET /api/applications/{id}/attachment` を呼び出す  
**Then** レスポンスは `200 OK` である  
**And** `Content-Type` は保存時の MIME タイプである  
**And** `Content-Disposition` に元ファイル名が含まれる  
**And** レスポンス body はアップロードしたファイル内容と一致する

---

### PURCHASE-008: 添付のない申請でダウンロード API は 404

**レイヤ**: API

**Given** 添付なしの物品購入申請（ID = `{id}`）が存在する  
**When** `GET /api/applications/{id}/attachment` を呼び出す  
**Then** レスポンスは `404 Not Found` である

---

### PURCHASE-009: 金額が負の申請は拒否される

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** `purchaseDetail.amount` が負の値で申請する  
**Then** レスポンスは `400 Bad Request`（`validation-error`）である

---

### PURCHASE-010: 休暇申請に multipart ファイルを付けられない

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**When** `type: LEAVE` の申請に `file` パートを含める  
**Then** レスポンスは `400 Bad Request` である
