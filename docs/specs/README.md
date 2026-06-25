# 機能別受け入れ条件

申請ワークフローシステムの振る舞いを **Given / When / Then** 形式で定義する。実装・テスト（API テスト / UI テスト / E2E）の判定基準とする。

## 関連ドキュメント

| ドキュメント | 役割 |
|--------------|------|
| [product-requirements.md](../product-requirements.md) | What（プロダクト要求） |
| [design.md](../design.md) | How（技術設計・Must 制約） |
| [api-spec.md](../api-spec.md) | API 正式定義 |

**優先順位**: 矛盾がある場合は `product-requirements.md` → `design.md` → `api-spec.md` → 本 specs の順で上位を正とする。

## 仕様ファイル一覧

| ファイル | 対象 |
|----------|------|
| [auth.md](./auth.md) | ログイン・ログアウト・セッション |
| [leave-application.md](./leave-application.md) | 休暇申請の作成・一覧・詳細 |
| [purchase-application.md](./purchase-application.md) | 物品購入申請・添付ファイル |
| [resubmit.md](./resubmit.md) | 否認後の再申請 |
| [approval.md](./approval.md) | 承認・否認 |
| [authorization.md](./authorization.md) | 認可・ロール制御 |
| [notification.md](./notification.md) | メール通知 |

## 記法

各シナリオは次の形式とする。

```
### {ID}: {タイトル}

**レイヤ**: API | UI | API+UI

**Given** {前提条件}
**When** {操作}
**Then** {期待結果}
```

- **API**: Rest Assured 等の API テストで検証可能
- **UI**: フロントエンドの画面操作で検証
- **API+UI**: 両方で同一の振る舞いを保証する

## 共通テストデータ（フィクスチャ）

Flyway シード等で投入する想定ユーザー。実装時に ID は環境依存のため、**loginId** を主キーとしてシナリオを記述する。

| loginId | displayName | isApprover | 用途 |
|---------|-------------|------------|------|
| `applicant1` | 申請 太郎 | `false` | 申請者専用ロールの代表 |
| `approver1` | 承認 花子 | `true` | 承認者 |
| `dual1` | 兼務 一郎 | `true` | 申請者兼承認者 |

- 初期パスワードはテスト環境用の既知値（例: `password`）とする。本番・検証では別途設定。
- `approver1` のメールアドレスは通知検証用に fake-smtp-server で受信可能な値とする。

## 列挙値（Must）

| 種別 | 値 |
|------|-----|
| ApplicationType | `LEAVE`, `PURCHASE` |
| ApplicationStatus | `PENDING`, `APPROVED`, `REJECTED` |
| ApprovalActionType | `APPROVE`, `REJECT`, `RESUBMIT` |
