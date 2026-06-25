# 通知 — 受け入れ条件

[product-requirements.md §5](../product-requirements.md#5-通知) / [api-spec.md §8](../api-spec.md#8-通知メール副作用)

**Must**: メール送信失敗で申請データのコミットをロールバックしない（API 成功と通知は切り離す）。

---

### NOTIFY-001: 新規申請時に承認者へメールが送られる

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**And** fake-smtp-server が起動している  
**When** `approver1` を承認者として `POST /api/applications` が成功する  
**Then** `approver1` のメールアドレス宛に通知メールが 1 通送信される  
**And** 件名に `[Workflow] 新規申請が届きました` が含まれる  
**And** 本文に申請 ID・申請種別・要約・申請者名が含まれる  
**And** 詳細リンク `{frontendBaseUrl}/approvals/{id}` が含まれる

---

### NOTIFY-002: 再申請時に承認者へメールが送られる

**レイヤ**: API

**Given** `REJECTED` の申請（ID = `{id}`）が存在する  
**When** `PUT /api/applications/{id}` による再申請が成功する  
**Then** 承認者宛に通知メールが 1 通送信される  
**And** 件名に `[Workflow] 申請が再提出されました` が含まれる

---

### NOTIFY-003: 承認時に申請者へメールが送られる

**レイヤ**: API

**Given** `PENDING` の申請が存在する  
**When** `POST /api/approvals/{id}/approve` が成功する  
**Then** 申請者のメールアドレス宛に通知メールが 1 通送信される  
**And** 件名に `[Workflow] 申請が承認されました` が含まれる  
**And** 詳細リンク `{frontendBaseUrl}/applications/{id}` が含まれる  
**And** 承認コメントがある場合は本文に含まれる

---

### NOTIFY-004: 否認時に申請者へメールが送られる（コメント含む）

**レイヤ**: API

**Given** `PENDING` の申請が存在する  
**When** コメント付きで `POST /api/approvals/{id}/reject` が成功する  
**Then** 申請者宛に通知メールが 1 通送信される  
**And** 件名に `[Workflow] 申請が否認されました` が含まれる  
**And** 否認コメントが本文に **必ず** 含まれる

---

### NOTIFY-005: メール送信失敗でも API は成功する

**レイヤ**: API

**Given** SMTP サーバーが利用不可、または送信が失敗する状態である  
**When** 申請作成・承認・否認・再申請の API がビジネスルール上成功する  
**Then** API レスポンスは成功（2xx）のままである  
**And** 申請データは DB にコミットされている  
**And** 送信失敗はログ等で記録される（リトライ方針は実装時に決定）

---

### NOTIFY-006: 通知対象外の操作ではメールを送らない

**レイヤ**: API

**Given** 申請が存在する  
**When** 次の操作のみを行う  
- `GET /api/applications`（一覧閲覧）  
- `GET /api/applications/{id}`（詳細閲覧）  
- `GET /api/approvals`（承認一覧閲覧）  
**Then** 新規の通知メールは送信されない
