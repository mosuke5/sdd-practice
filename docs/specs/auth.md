# 認証 — 受け入れ条件

[api-spec.md §6.1](../api-spec.md#61-認証) / [design.md §6](../design.md#6-認証セキュリティ)

---

### AUTH-001: 正しい認証情報でログインできる

**レイヤ**: API+UI

**Given** 未ログイン状態である  
**And** ユーザー `applicant1` が存在し、パスワードが正しく設定されている  
**When** `POST /api/auth/login` に `{ "loginId": "applicant1", "password": "password" }` を送信する  
**Then** レスポンスは `200 OK` である  
**And** レスポンス body に `user.loginId` が `applicant1` である  
**And** レスポンスヘッダにセッション Cookie（`Set-Cookie`）が含まれる  
**And** UI ではログイン画面から申請一覧等の認証後画面へ遷移する

---

### AUTH-002: 誤ったパスワードでログインできない

**レイヤ**: API+UI

**Given** 未ログイン状態である  
**When** `POST /api/auth/login` に誤ったパスワードを送信する  
**Then** レスポンスは `401 Unauthorized` である  
**And** レスポンスは RFC 7807 Problem Details（`application/problem+json`）形式である  
**And** セッション Cookie は発行されない  
**And** UI では `detail` の内容をユーザー向けエラーメッセージとして表示する

---

### AUTH-003: ログイン中のユーザー情報を取得できる

**レイヤ**: API

**Given** `applicant1` でログイン済みである  
**When** `GET /api/auth/me` を Cookie 付きで呼び出す  
**Then** レスポンスは `200 OK` である  
**And** `loginId`, `displayName`, `email`, `isApprover` が返る

---

### AUTH-004: 未認証で保護 API にアクセスできない

**レイヤ**: API+UI

**Given** 未ログイン状態である  
**When** `GET /api/applications` を Cookie なしで呼び出す  
**Then** レスポンスは `401 Unauthorized` である  
**And** UI では未認証時に保護画面へ直接アクセスした場合、ログイン画面へリダイレクトする

---

### AUTH-005: ログアウトでセッションが無効化される

**レイヤ**: API+UI

**Given** `applicant1` でログイン済みである  
**When** `POST /api/auth/logout` を呼び出す  
**Then** レスポンスは `204 No Content` である  
**And** 以降、同一 Cookie で `GET /api/auth/me` を呼ぶと `401 Unauthorized` となる  
**And** UI ではログイン画面へ遷移する

---

### AUTH-006: ログイン API は未認証でも利用できる

**レイヤ**: API

**Given** 未ログイン状態である  
**When** `POST /api/auth/login` を呼び出す  
**Then** `401` ではなく、認証結果に応じた `200` または `401`（認証失敗）が返る（エンドポイント自体は未認証アクセス可）
