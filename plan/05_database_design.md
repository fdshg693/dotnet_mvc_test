# ブログシステム データベース詳細設計

## 1. ER図

```
┌─────────────────────┐
│   AspNetUsers       │
│ (Identity標準)      │
├─────────────────────┤
│ Id (PK)             │
│ UserName            │
│ Email               │
│ PasswordHash        │
│ ...                 │
└──────────┬──────────┘
           │
           │ 1
           │
           │ *
┌──────────┴──────────┐         ┌─────────────────────┐
│   Articles          │    *    │   Categories        │
├─────────────────────┤ ────┬─→ ├─────────────────────┤
│ Id (PK)             │     1   │ Id (PK)             │
│ Title               │         │ Name                │
│ Slug (UK)           │         │ Slug (UK)           │
│ Content             │         │ Description         │
│ Summary             │         │ CreatedAt           │
│ FeaturedImageUrl    │         └─────────────────────┘
│ CategoryId (FK)     │
│ AuthorId (FK)       │
│ IsPublished         │         ┌─────────────────────┐
│ PublishedAt         │    *    │   Tags              │
│ CreatedAt           │ ────┬─→ ├─────────────────────┤
│ UpdatedAt           │     │   │ Id (PK)             │
│ IsDeleted           │     │   │ Name                │
└──────────┬──────────┘     │   │ Slug (UK)           │
           │                │   │ CreatedAt           │
           │ 1              │   └─────────────────────┘
           │                │
           │ *              │
┌──────────┴──────────┐     │
│   Comments          │     │   ┌─────────────────────┐
├─────────────────────┤     │   │   ArticleTags       │
│ Id (PK)             │     └─→ ├─────────────────────┤
│ ArticleId (FK)      │     *   │ ArticleId (PK, FK)  │
│ AuthorName          │         │ TagId (PK, FK)      │
│ AuthorEmail         │         └─────────────────────┘
│ Content             │
│ IsApproved          │
│ CreatedAt           │
└─────────────────────┘
```

## 2. テーブル定義詳細

### 2.1 Articles（記事テーブル）

```sql
CREATE TABLE Articles (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Title NVARCHAR(200) NOT NULL,
    Slug NVARCHAR(200) NOT NULL UNIQUE,
    Content NVARCHAR(MAX) NOT NULL,
    Summary NVARCHAR(500),
    FeaturedImageUrl NVARCHAR(500),
    CategoryId INT NOT NULL,
    AuthorId NVARCHAR(450) NOT NULL,
    IsPublished BIT NOT NULL DEFAULT 0,
    PublishedAt DATETIME2,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    IsDeleted BIT NOT NULL DEFAULT 0,
    
    CONSTRAINT FK_Articles_Categories FOREIGN KEY (CategoryId) 
        REFERENCES Categories(Id),
    CONSTRAINT FK_Articles_Users FOREIGN KEY (AuthorId) 
        REFERENCES AspNetUsers(Id)
);

CREATE INDEX IX_Articles_CategoryId ON Articles(CategoryId);
CREATE INDEX IX_Articles_AuthorId ON Articles(AuthorId);
CREATE INDEX IX_Articles_PublishedAt ON Articles(PublishedAt DESC);
CREATE INDEX IX_Articles_IsPublished_IsDeleted ON Articles(IsPublished, IsDeleted);
```

#### カラム説明
| カラム名 | 型 | NULL | デフォルト | 説明 |
|---------|------|------|-----------|------|
| Id | INT | NO | IDENTITY | 主キー、自動採番 |
| Title | NVARCHAR(200) | NO | - | 記事タイトル |
| Slug | NVARCHAR(200) | NO | - | URLスラッグ（一意） |
| Content | NVARCHAR(MAX) | NO | - | 記事本文（Markdown形式） |
| Summary | NVARCHAR(500) | YES | NULL | 記事の概要・説明文 |
| FeaturedImageUrl | NVARCHAR(500) | YES | NULL | アイキャッチ画像のURL |
| CategoryId | INT | NO | - | カテゴリID（外部キー） |
| AuthorId | NVARCHAR(450) | NO | - | 投稿者ID（外部キー） |
| IsPublished | BIT | NO | 0 | 公開フラグ（0:下書き, 1:公開） |
| PublishedAt | DATETIME2 | YES | NULL | 公開日時 |
| CreatedAt | DATETIME2 | NO | GETUTCDATE() | 作成日時 |
| UpdatedAt | DATETIME2 | NO | GETUTCDATE() | 更新日時 |
| IsDeleted | BIT | NO | 0 | 削除フラグ（論理削除） |

---

### 2.2 Categories（カテゴリテーブル）

```sql
CREATE TABLE Categories (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Slug NVARCHAR(100) NOT NULL UNIQUE,
    Description NVARCHAR(500),
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);
```

#### カラム説明
| カラム名 | 型 | NULL | デフォルト | 説明 |
|---------|------|------|-----------|------|
| Id | INT | NO | IDENTITY | 主キー、自動採番 |
| Name | NVARCHAR(100) | NO | - | カテゴリ名 |
| Slug | NVARCHAR(100) | NO | - | URLスラッグ（一意） |
| Description | NVARCHAR(500) | YES | NULL | カテゴリの説明 |
| CreatedAt | DATETIME2 | NO | GETUTCDATE() | 作成日時 |

---

### 2.3 Tags（タグテーブル）

```sql
CREATE TABLE Tags (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(50) NOT NULL,
    Slug NVARCHAR(50) NOT NULL UNIQUE,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);
```

#### カラム説明
| カラム名 | 型 | NULL | デフォルト | 説明 |
|---------|------|------|-----------|------|
| Id | INT | NO | IDENTITY | 主キー、自動採番 |
| Name | NVARCHAR(50) | NO | - | タグ名 |
| Slug | NVARCHAR(50) | NO | - | URLスラッグ（一意） |
| CreatedAt | DATETIME2 | NO | GETUTCDATE() | 作成日時 |

---

### 2.4 ArticleTags（記事-タグ中間テーブル）

```sql
CREATE TABLE ArticleTags (
    ArticleId INT NOT NULL,
    TagId INT NOT NULL,
    
    PRIMARY KEY (ArticleId, TagId),
    CONSTRAINT FK_ArticleTags_Articles FOREIGN KEY (ArticleId) 
        REFERENCES Articles(Id) ON DELETE CASCADE,
    CONSTRAINT FK_ArticleTags_Tags FOREIGN KEY (TagId) 
        REFERENCES Tags(Id) ON DELETE CASCADE
);

CREATE INDEX IX_ArticleTags_TagId ON ArticleTags(TagId);
```

#### カラム説明
| カラム名 | 型 | NULL | 説明 |
|---------|------|------|------|
| ArticleId | INT | NO | 記事ID（外部キー、複合主キー） |
| TagId | INT | NO | タグID（外部キー、複合主キー） |

---

### 2.5 Comments（コメントテーブル）

```sql
CREATE TABLE Comments (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    ArticleId INT NOT NULL,
    AuthorName NVARCHAR(100) NOT NULL,
    AuthorEmail NVARCHAR(256) NOT NULL,
    Content NVARCHAR(1000) NOT NULL,
    IsApproved BIT NOT NULL DEFAULT 0,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    
    CONSTRAINT FK_Comments_Articles FOREIGN KEY (ArticleId) 
        REFERENCES Articles(Id) ON DELETE CASCADE
);

CREATE INDEX IX_Comments_ArticleId ON Comments(ArticleId);
CREATE INDEX IX_Comments_IsApproved ON Comments(IsApproved);
CREATE INDEX IX_Comments_CreatedAt ON Comments(CreatedAt DESC);
```

#### カラム説明
| カラム名 | 型 | NULL | デフォルト | 説明 |
|---------|------|------|-----------|------|
| Id | INT | NO | IDENTITY | 主キー、自動採番 |
| ArticleId | INT | NO | - | 記事ID（外部キー） |
| AuthorName | NVARCHAR(100) | NO | - | コメント投稿者名 |
| AuthorEmail | NVARCHAR(256) | NO | - | コメント投稿者メール |
| Content | NVARCHAR(1000) | NO | - | コメント内容 |
| IsApproved | BIT | NO | 0 | 承認フラグ（0:未承認, 1:承認済み） |
| CreatedAt | DATETIME2 | NO | GETUTCDATE() | 作成日時 |

---

### 2.6 AspNetUsers（ユーザーテーブル - Identity）

ASP.NET Core Identityの標準テーブルを使用します。
拡張が必要な場合は、ApplicationUserクラスで追加プロパティを定義します。

```csharp
// 拡張例（必要に応じて）
public class ApplicationUser : IdentityUser
{
    public string DisplayName { get; set; }
    public string Bio { get; set; }
    public string AvatarUrl { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

---

## 3. インデックス戦略

### 3.1 パフォーマンス重視のインデックス

```sql
-- 記事一覧の高速化（公開済み記事を日付順で取得）
CREATE INDEX IX_Articles_IsPublished_IsDeleted_PublishedAt 
    ON Articles(IsPublished, IsDeleted, PublishedAt DESC);

-- カテゴリ別記事一覧の高速化
CREATE INDEX IX_Articles_CategoryId_IsPublished_PublishedAt 
    ON Articles(CategoryId, IsPublished, PublishedAt DESC);

-- タグ別記事一覧の高速化
CREATE INDEX IX_ArticleTags_TagId_ArticleId 
    ON ArticleTags(TagId, ArticleId);

-- 検索機能の高速化（フルテキストインデックス）
CREATE FULLTEXT INDEX ON Articles(Title, Content) 
    KEY INDEX PK_Articles;
```

---

## 4. 初期データ（Seed Data）

### 4.1 カテゴリ初期データ

```csharp
var categories = new[]
{
    new Category { Id = 1, Name = "技術", Slug = "tech", Description = "技術関連の記事" },
    new Category { Id = 2, Name = "日記", Slug = "diary", Description = "日常の出来事" },
    new Category { Id = 3, Name = "お知らせ", Slug = "news", Description = "サイトのお知らせ" }
};
```

### 4.2 管理者ユーザー初期データ

```csharp
var adminUser = new ApplicationUser
{
    UserName = "admin",
    Email = "admin@example.com",
    EmailConfirmed = true
};
// パスワード: Admin@123
```

### 4.3 サンプル記事データ

```csharp
var sampleArticle = new Article
{
    Title = "ようこそブログへ",
    Slug = "welcome",
    Content = "# ようこそ\n\nこのブログへようこそ！",
    Summary = "最初の記事です",
    CategoryId = 3,
    AuthorId = adminUser.Id,
    IsPublished = true,
    PublishedAt = DateTime.UtcNow,
    CreatedAt = DateTime.UtcNow,
    UpdatedAt = DateTime.UtcNow
};
```

---

## 5. マイグレーション手順

### 5.1 マイグレーションコマンド

```powershell
# 初期マイグレーション作成
dotnet ef migrations add InitialCreate

# データベース更新
dotnet ef database update

# マイグレーション削除（必要な場合）
dotnet ef migrations remove
```

### 5.2 マイグレーションファイル構成

```
Migrations/
├── 20231108000001_InitialCreate.cs
├── 20231108000001_InitialCreate.Designer.cs
├── 20231108000002_AddArticlesTables.cs
├── 20231108000003_AddCommentsTables.cs
└── ApplicationDbContextModelSnapshot.cs
```

---

## 6. データベース運用

### 6.1 バックアップ計画
- **頻度**: 日次自動バックアップ
- **保持期間**: 30日間
- **方法**: SQL Server自動バックアップジョブ

### 6.2 メンテナンス
- **インデックス再構築**: 週次
- **統計情報更新**: 日次
- **不要データ削除**: 論理削除から90日経過後

### 6.3 監視項目
- テーブルサイズ
- インデックスフラグメンテーション
- 実行時間の長いクエリ
