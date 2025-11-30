# 管理者向け2段階認証（TOTP）実装方針

**作成日:** 2025-11-30  
**ステータス:** 計画中

---

## 1. 概要

### 1.1 目的
管理者（Administrator ロール）アカウントに対して、TOTP（Time-based One-Time Password）ベースの2段階認証を強制し、セキュリティを強化する。

### 1.2 対象ユーザー
- **Administrator** ロールを持つユーザー（現在: `admin@example.com`）
- 一般ユーザー（User ロール）は任意設定

### 1.3 技術選定
| 項目 | 選択 | 理由 |
|------|------|------|
| 認証方式 | TOTP (RFC 6238) | Google Authenticator、Microsoft Authenticator 等で広くサポート |
| QRコード生成 | QRCoder ライブラリ | .NET 対応、軽量、MIT ライセンス |
| 秘密鍵管理 | ASP.NET Identity 標準機能 | `UserManager.GetAuthenticatorKeyAsync()` を使用 |

---

## 2. 機能要件

### 2.1 管理者への2FA強制
1. **ログイン時チェック**: 管理者が2FAを未設定の場合、2FA設定ページへ強制リダイレクト
2. **管理画面アクセス制御**: 2FA未設定の管理者は管理画面（`/admin/*`）にアクセス不可
3. **設定無効化防止**: 管理者は自身の2FAを無効化できない

### 2.2 2FA設定フロー
```
[ログイン] → [2FA未設定?] → [Yes] → [2FA設定ページ]
                ↓ No                       ↓
           [通常遷移]              [QRコード表示]
                                           ↓
                                   [認証アプリでスキャン]
                                           ↓
                                   [確認コード入力]
                                           ↓
                                   [リカバリーコード表示]
                                           ↓
                                   [設定完了]
```

### 2.3 ログインフロー（2FA設定済み）
```
[メール/パスワード入力] → [認証成功] → [2FAページへリダイレクト]
                                              ↓
                                      [6桁コード入力]
                                              ↓
                                      [検証成功] → [ログイン完了]
```

### 2.4 リカバリーコード
- 2FA設定時に10個のリカバリーコードを生成・表示
- 各コードは1回限り使用可能
- 認証アプリにアクセスできない場合の代替手段

---

## 3. 技術設計

### 3.1 必要なNuGetパッケージ
```xml
<PackageReference Include="QRCoder" Version="1.6.0" />
```

### 3.2 新規作成ファイル

#### Controllers
| ファイル | 説明 |
|---------|------|
| `Controllers/TwoFactorController.cs` | 2FA設定・検証コントローラー |

#### Views
| ファイル | 説明 |
|---------|------|
| `Views/TwoFactor/Setup.cshtml` | QRコード表示・初期設定画面 |
| `Views/TwoFactor/Verify.cshtml` | ログイン時の確認コード入力画面 |
| `Views/TwoFactor/RecoveryCodes.cshtml` | リカバリーコード表示画面 |

#### ViewModels
| ファイル | 説明 |
|---------|------|
| `Models/ViewModels/TwoFactor/SetupTwoFactorViewModel.cs` | 設定画面用 |
| `Models/ViewModels/TwoFactor/VerifyTwoFactorViewModel.cs` | 検証画面用 |
| `Models/ViewModels/TwoFactor/RecoveryCodesViewModel.cs` | リカバリーコード用 |

#### Services
| ファイル | 説明 |
|---------|------|
| `Services/ITwoFactorService.cs` | 2FA サービスインターフェース |
| `Services/TwoFactorService.cs` | QRコード生成等のビジネスロジック |

#### Middleware / Filters
| ファイル | 説明 |
|---------|------|
| `Filters/RequireTwoFactorAttribute.cs` | 管理者2FA強制用のアクションフィルター |

### 3.3 既存ファイルの変更

| ファイル | 変更内容 |
|---------|---------|
| `Program.cs` | サービス登録、2FA設定の追加 |
| `Controllers/AccountController.cs` | `RequiresTwoFactor` ハンドリングの実装 |
| `Controllers/Admin/DashboardController.cs` | 2FA強制フィルター適用 |
| `Controllers/Admin/ArticleController.cs` | 2FA強制フィルター適用 |
| `Views/Shared/_Layout.cshtml` | 2FA設定リンクの追加（任意） |

### 3.4 データベース変更
**マイグレーション不要** - ASP.NET Identity の既存カラムを使用:
- `AspNetUsers.TwoFactorEnabled` - 2FA有効フラグ
- `AspNetUsers.AuthenticatorKey` - TOTP秘密鍵（`UserTokens`テーブル経由）

---

## 4. 実装詳細

### 4.1 QRコード生成ロジック
```csharp
// otpauth://totp/{Issuer}:{Email}?secret={Key}&issuer={Issuer}
var otpauthUri = $"otpauth://totp/{Uri.EscapeDataString(issuer)}:{Uri.EscapeDataString(email)}?secret={key}&issuer={Uri.EscapeDataString(issuer)}";
```

### 4.2 管理者2FA強制の実装方法

#### 方法A: カスタムアクションフィルター（推奨）
```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class RequireAdminTwoFactorAttribute : ActionFilterAttribute
{
    public override async Task OnActionExecutionAsync(
        ActionExecutingContext context, 
        ActionExecutionDelegate next)
    {
        // 管理者で2FA未設定の場合、設定ページへリダイレクト
    }
}
```

#### 方法B: ミドルウェア
管理画面全体（`/admin/*`）に適用する場合に有効だが、粒度が粗い。

**選択:** 方法A（フィルター）を採用 - 柔軟性と明示性のバランスが良い

### 4.3 セキュリティ考慮事項

| 項目 | 対策 |
|------|------|
| 秘密鍵の保護 | Identity UserTokens テーブルに暗号化保存 |
| ブルートフォース | レート制限（連続5回失敗でロックアウト） |
| リカバリーコード | ハッシュ化して保存、使用後は無効化 |
| QRコード | サーバーサイド生成、セッション紐付け |

---

## 5. UI/UX設計

### 5.1 2FA設定画面
```
┌─────────────────────────────────────────────────────────┐
│  2段階認証の設定                                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. 認証アプリをインストール                              │
│     Google Authenticator または Microsoft Authenticator  │
│                                                          │
│  2. 下のQRコードをスキャン                                │
│     ┌─────────────┐                                      │
│     │  [QRコード]  │                                      │
│     └─────────────┘                                      │
│     手動入力用キー: XXXX XXXX XXXX XXXX                  │
│                                                          │
│  3. 確認コードを入力                                      │
│     ┌─────────────┐                                      │
│     │             │  [確認] ボタン                        │
│     └─────────────┘                                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 5.2 ログイン時2FA入力画面
```
┌─────────────────────────────────────────────────────────┐
│  2段階認証                                               │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  認証アプリに表示されている6桁のコードを入力してください   │
│                                                          │
│     ┌─────────────┐                                      │
│     │             │                                      │
│     └─────────────┘                                      │
│                                                          │
│     [ログイン] ボタン                                     │
│                                                          │
│     リカバリーコードを使用                                │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 6. 実装手順

### Phase 1: 基盤構築
1. [ ] QRCoder パッケージのインストール
2. [ ] TwoFactorService の作成（インターフェース＋実装）
3. [ ] ViewModels の作成
4. [ ] Program.cs へのサービス登録

### Phase 2: 2FA設定機能
5. [ ] TwoFactorController の作成
6. [ ] Setup ビューの作成（QRコード表示）
7. [ ] 確認コード検証ロジックの実装
8. [ ] リカバリーコード生成・表示

### Phase 3: ログイン統合
9. [ ] AccountController の2FAハンドリング実装
10. [ ] Verify ビューの作成
11. [ ] リカバリーコードでのログイン対応

### Phase 4: 管理者強制
12. [ ] RequireAdminTwoFactorAttribute の作成
13. [ ] 管理コントローラーへのフィルター適用
14. [ ] 未設定時のリダイレクトロジック

### Phase 5: テスト・仕上げ
15. [ ] 手動テスト（設定→ログイン→リカバリー）
16. [ ] エラーハンドリングの確認
17. [ ] UIの日本語化確認

---

## 7. テスト計画

### 7.1 機能テスト
| # | シナリオ | 期待結果 |
|---|---------|---------|
| 1 | 管理者が2FA未設定でログイン | 設定ページへリダイレクト |
| 2 | QRコードスキャン後、正しいコード入力 | 2FA有効化成功 |
| 3 | 2FA設定済み管理者がログイン | 確認コード入力画面表示 |
| 4 | 正しい確認コードでログイン | ログイン成功 |
| 5 | 誤った確認コードでログイン | エラー表示、再入力 |
| 6 | リカバリーコードでログイン | ログイン成功、コード無効化 |
| 7 | 一般ユーザーが2FAなしでログイン | 通常通りログイン成功 |

### 7.2 セキュリティテスト
| # | シナリオ | 期待結果 |
|---|---------|---------|
| 1 | 2FA未設定管理者が `/admin/` 直接アクセス | 設定ページへリダイレクト |
| 2 | 連続5回誤コード入力 | アカウントロックアウト |
| 3 | 使用済みリカバリーコードの再利用 | 拒否 |

---

## 8. 参考資料

- [ASP.NET Core Identity 2FA ドキュメント](https://learn.microsoft.com/aspnet/core/security/authentication/identity-enable-qrcodes)
- [RFC 6238 - TOTP](https://datatracker.ietf.org/doc/html/rfc6238)
- [QRCoder GitHub](https://github.com/codebude/QRCoder)

---

## 9. 今後の拡張（スコープ外）

- SMS/メールによる2FA（代替手段）
- ハードウェアキー（FIDO2/WebAuthn）対応
- バックアップ用の信頼済みデバイス登録
- 2FA設定のバルク管理（管理者による強制リセット）
