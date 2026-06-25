# 再申請 — 受け入れ条件

[product-requirements.md §4.1.3](../product-requirements.md#413-否認後の修正再申請) / [api-spec.md §5, §6.3](../api-spec.md#53-再申請時の履歴)

**Must**: 否認後の再申請は **同一 `Application` レコードを更新**する。新規レコードは作成しない。申請 ID・URL は変わらない。

---

### RESUBMIT-001: 否認された休暇申請を再申請できる

**レイヤ**: API+UI

**Given** `applicant1` の休暇申請（ID = `{id}`）が `REJECTED` である  
**When** `PUT /api/applications/{id}` で内容を修正して送信する  
**Then** レスポンスは `200 OK` である  
**And** `status` は `PENDING` に戻る  
**And** 申請 ID は再申請前後で同一である  
**And** `approvalActions` に `RESUBMIT` が 1 件追加される  
**And** 過去の `REJECT` 履歴は残る  
**And** UI では `/applications/{id}` の URL が変わらない

---

### RESUBMIT-002: 否認された物品購入申請を再申請できる（添付維持）

**レイヤ**: API

**Given** 添付付き物品購入申請（ID = `{id}`）が `REJECTED` である  
**When** `PUT /api/applications/{id}` を `file` パートなしで送信する  
**Then** レスポンスは `200 OK` である  
**And** `status` は `PENDING` である  
**And** 既存の `attachment` メタデータが維持される

---

### RESUBMIT-003: 再申請時に添付ファイルを差し替えできる

**レイヤ**: API

**Given** 添付付き物品購入申請（ID = `{id}`）が `REJECTED` である  
**When** `PUT /api/applications/{id}` に新しい有効な `file` パートを含めて送信する  
**Then** レスポンスは `200 OK` である  
**And** `attachment` が新ファイルのメタデータに更新される  
**And** 旧オブジェクトストレージ上のファイルは削除される

---

### RESUBMIT-004: 再申請時に添付を削除のみ（添付なしへ変更）はできない

**レイヤ**: API+UI

**Given** 添付付き物品購入申請（ID = `{id}`）が `REJECTED` である  
**When** `file` パートなしで申請内容のみ更新する（添付削除を意図）  
**Then** 添付は **維持** される（初版では明示的削除不可）  
**And** 添付を外す専用 UI / API は提供しない

---

### RESUBMIT-005: PENDING の申請は再申請（更新）できない

**レイヤ**: API+UI

**Given** 休暇申請（ID = `{id}`）が `PENDING` である  
**When** `PUT /api/applications/{id}` を呼び出す  
**Then** レスポンスは `409 Conflict` である  
**And** `detail` に審査中の申請は更新できない旨が含まれる  
**And** UI では `PENDING` 申請に再申請フォームを表示しない

---

### RESUBMIT-006: APPROVED の申請は再申請・再編集できない

**レイヤ**: API+UI

**Given** 休暇申請（ID = `{id}`）が `APPROVED` である  
**When** `PUT /api/applications/{id}` を呼び出す  
**Then** レスポンスは `409 Conflict` である  
**And** `detail` に承認済みの申請は更新できない旨が含まれる  
**And** UI では `APPROVED` 申請を読み取り専用で表示する

---

### RESUBMIT-007: 再申請時に申請種別は変更できない

**レイヤ**: API

**Given** 休暇申請（ID = `{id}`）が `REJECTED` である  
**When** `PUT /api/applications/{id}` で `type: PURCHASE` に変更して送信する  
**Then** レスポンスは `400 Bad Request` である

---

### RESUBMIT-008: 他人の申請は再申請できない

**レイヤ**: API

**Given** `applicant1` の申請（ID = `{id}`）が `REJECTED` である  
**When** 別ユーザー `dual1`（申請者ではない）が `PUT /api/applications/{id}` を呼び出す  
**Then** レスポンスは `403 Forbidden` である

---

### RESUBMIT-009: RESUBMIT の ApprovalAction にコメントは任意

**レイヤ**: API

**Given** 否認済み申請が存在する  
**When** コメントなしで再申請に成功する  
**Then** `RESUBMIT` の `ApprovalAction.comment` は `null` でもよい
