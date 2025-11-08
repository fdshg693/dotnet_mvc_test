# ブログシステム アーキテクチャ設計

## 1. システム構成

### 1.1 全体構成図
```
┌─────────────────────────────────────────┐
│          クライアント（ブラウザ）          │
└─────────────────────────────────────────┘
                    │
                    │ HTTPS
                    ▼
┌─────────────────────────────────────────┐
│         ASP.NET Core MVC アプリ          │
│  ┌─────────────────────────────────┐    │
│  │      Presentation Layer         │    │
│  │  - Controllers                  │    │
│  │  - Views (Razor)                │    │
│  │  - ViewModels                   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │      Business Logic Layer       │    │
│  │  - Services                     │    │
│  │  - Business Rules               │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │      Data Access Layer          │    │
│  │  - Entity Framework Core        │    │
│  │  - Repositories                 │    │
│  │  - DbContext                    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│          データベース（SQL Server）        │
└─────────────────────────────────────────┘
```

### 1.2 レイヤー構成

#### Presentation Layer（プレゼンテーション層）
- **Controllers**: HTTPリクエストの受付とレスポンス返却
- **Views**: Razorビューによる画面表示
- **ViewModels**: 画面表示用のデータ転送オブジェクト

#### Business Logic Layer（ビジネスロジック層）
- **Services**: ビジネスロジックの実装
- **Validators**: 入力検証ロジック
- **Mappers**: エンティティとViewModelの変換

#### Data Access Layer（データアクセス層）
- **Entities**: データベーステーブルに対応するモデル
- **DbContext**: Entity Framework Coreのコンテキスト
- **Repositories**: データアクセスの抽象化（オプション）

## 2. ディレクトリ構造

```
dotnet_mvc_test/
├── Controllers/
│   ├── HomeController.cs           # 既存
│   ├── BlogController.cs           # 記事閲覧
│   ├── Admin/
│   │   ├── ArticleController.cs    # 記事管理
│   │   ├── CategoryController.cs   # カテゴリ管理
│   │   ├── TagController.cs        # タグ管理
│   │   └── CommentController.cs    # コメント管理
│   └── AccountController.cs        # 認証
├── Models/
│   ├── Entities/                   # データベースエンティティ
│   │   ├── Article.cs
│   │   ├── Category.cs
│   │   ├── Tag.cs
│   │   ├── Comment.cs
│   │   └── ApplicationUser.cs
│   ├── ViewModels/                 # 画面用ViewModel
│   │   ├── Blog/
│   │   ├── Admin/
│   │   └── Account/
│   └── ErrorViewModel.cs           # 既存
├── Services/
│   ├── Interfaces/
│   │   ├── IArticleService.cs
│   │   ├── ICategoryService.cs
│   │   ├── ITagService.cs
│   │   └── ICommentService.cs
│   └── Implementations/
│       ├── ArticleService.cs
│       ├── CategoryService.cs
│       ├── TagService.cs
│       └── CommentService.cs
├── Data/
│   ├── ApplicationDbContext.cs     # EF Core DbContext
│   ├── Migrations/                 # マイグレーションファイル
│   └── Seed/                       # 初期データ
│       └── SeedData.cs
├── Views/
│   ├── Shared/
│   │   ├── _Layout.cshtml          # 既存
│   │   └── _AdminLayout.cshtml     # 管理画面用レイアウト
│   ├── Home/                       # 既存
│   ├── Blog/
│   │   ├── Index.cshtml            # 記事一覧
│   │   ├── Detail.cshtml           # 記事詳細
│   │   ├── Category.cshtml         # カテゴリ別一覧
│   │   ├── Tag.cshtml              # タグ別一覧
│   │   └── Search.cshtml           # 検索結果
│   ├── Admin/
│   │   ├── Dashboard/
│   │   ├── Article/
│   │   ├── Category/
│   │   ├── Tag/
│   │   └── Comment/
│   └── Account/
│       ├── Login.cshtml
│       └── Logout.cshtml
├── wwwroot/
│   ├── css/
│   │   ├── site.css                # 既存
│   │   └── admin.css               # 管理画面用CSS
│   ├── js/
│   │   ├── site.js                 # 既存
│   │   └── admin.js                # 管理画面用JS
│   ├── images/                     # アップロード画像
│   └── lib/                        # 既存（Bootstrap, jQuery等）
├── Utilities/
│   ├── MarkdownHelper.cs           # Markdown変換
│   └── SlugHelper.cs               # URLスラッグ生成
├── Program.cs                      # 既存
├── appsettings.json                # 既存
└── dotnet_mvc_test.csproj          # 既存
```

## 3. データベース設計

### 3.1 テーブル構成

#### Articles（記事テーブル）
| カラム名 | 型 | 説明 |
|---------|------|------|
| Id | int | 主キー |
| Title | nvarchar(200) | タイトル |
| Slug | nvarchar(200) | URLスラッグ |
| Content | nvarchar(max) | 本文（Markdown） |
| Summary | nvarchar(500) | 概要 |
| FeaturedImageUrl | nvarchar(500) | アイキャッチ画像URL |
| CategoryId | int | カテゴリID（外部キー） |
| AuthorId | nvarchar(450) | 投稿者ID（外部キー） |
| IsPublished | bit | 公開フラグ |
| PublishedAt | datetime2 | 公開日時 |
| CreatedAt | datetime2 | 作成日時 |
| UpdatedAt | datetime2 | 更新日時 |
| IsDeleted | bit | 削除フラグ |

#### Categories（カテゴリテーブル）
| カラム名 | 型 | 説明 |
|---------|------|------|
| Id | int | 主キー |
| Name | nvarchar(100) | カテゴリ名 |
| Slug | nvarchar(100) | URLスラッグ |
| Description | nvarchar(500) | 説明 |
| CreatedAt | datetime2 | 作成日時 |

#### Tags（タグテーブル）
| カラム名 | 型 | 説明 |
|---------|------|------|
| Id | int | 主キー |
| Name | nvarchar(50) | タグ名 |
| Slug | nvarchar(50) | URLスラッグ |
| CreatedAt | datetime2 | 作成日時 |

#### ArticleTags（記事-タグ中間テーブル）
| カラム名 | 型 | 説明 |
|---------|------|------|
| ArticleId | int | 記事ID（外部キー） |
| TagId | int | タグID（外部キー） |

#### Comments（コメントテーブル）
| カラム名 | 型 | 説明 |
|---------|------|------|
| Id | int | 主キー |
| ArticleId | int | 記事ID（外部キー） |
| AuthorName | nvarchar(100) | 投稿者名 |
| AuthorEmail | nvarchar(256) | 投稿者メール |
| Content | nvarchar(1000) | コメント内容 |
| IsApproved | bit | 承認フラグ |
| CreatedAt | datetime2 | 作成日時 |

#### AspNetUsers（ユーザーテーブル - Identity）
- ASP.NET Core Identityの標準テーブルを使用

### 3.2 リレーション
```
Categories 1 ─────── * Articles
Articles * ─────── * Tags (ArticleTags経由)
Articles 1 ─────── * Comments
AspNetUsers 1 ─────── * Articles
```

## 4. セキュリティ設計

### 4.1 認証
- ASP.NET Core Identityを使用
- Cookie認証
- パスワードポリシー：最低8文字、大文字・小文字・数字を含む

### 4.2 認可
- ロールベース認可（Administrator, User）
- 管理画面は[Authorize(Roles = "Administrator")]で保護

### 4.3 データ保護
- Entity Framework CoreのパラメータクエリによるSQLインジェクション対策
- Razor自動エスケープによるXSS対策
- AntiForgeryTokenによるCSRF対策

## 5. パフォーマンス最適化

### 5.1 キャッシュ戦略
- 記事一覧：メモリキャッシュ（5分）
- カテゴリ・タグ一覧：メモリキャッシュ（30分）

### 5.2 データベース最適化
- 適切なインデックスの設定
- Eager Loading / Lazy Loadingの使い分け
- ページング処理の実装

## 6. エラーハンドリング

- グローバルエラーハンドラーの実装
- カスタムエラーページ（404, 500）
- ログ出力（Serilog使用）
