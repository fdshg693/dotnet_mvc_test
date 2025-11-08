# ブログシステム 使用予定技術

## 1. 開発環境

### 1.1 フレームワーク・ランタイム
- **.NET 9.0**
  - 現在の最新LTSバージョン
  - ASP.NET Core MVC 9.0

### 1.2 開発ツール
- **Visual Studio 2022** または **Visual Studio Code**
- **SQL Server Management Studio (SSMS)** - データベース管理
- **Git** - バージョン管理

## 2. バックエンド技術

### 2.1 フレームワーク・ライブラリ

#### コア機能
- **ASP.NET Core MVC 9.0**
  - MVCパターンによるWeb開発
  - Razorビューエンジン

#### データアクセス
- **Entity Framework Core 9.0**
  - Code First アプローチ
  - マイグレーション機能
  - LINQ to Entities

#### 認証・認可
- **ASP.NET Core Identity**
  - ユーザー管理
  - ロールベース認可
  - パスワードハッシュ化

#### その他
- **Markdig** (v0.37.0以降)
  - Markdown → HTML変換
  - GitHub Flavored Markdown対応
  - 拡張機能サポート

- **Serilog** (v3.1.0以降)
  - 構造化ログ
  - ファイル・コンソール出力
  - ASP.NET Core統合

- **AutoMapper** (v12.0以降)
  - オブジェクトマッピング
  - Entity ↔ ViewModel変換

- **FluentValidation** (v11.9以降)
  - モデル検証
  - カスタムバリデーションルール

### 2.2 データベース
- **SQL Server 2019以降** または **SQL Server Express**
  - リレーショナルデータベース
  - Entity Framework Coreと連携

代替案：
- **SQLite** - 開発環境用（軽量）
- **PostgreSQL** - 本番環境用（オープンソース）

## 3. フロントエンド技術

### 3.1 UIフレームワーク
- **Bootstrap 5.3**（既にテンプレートに含まれている）
  - レスポンシブデザイン
  - グリッドシステム
  - UIコンポーネント

### 3.2 JavaScript
- **jQuery 3.7** (既存)
  - DOM操作
  - Ajax通信
  - jQueryプラグイン利用

- **jQuery Validation** (既存)
  - クライアント側バリデーション

- **Chart.js** (v4.4以降) - 予定
  - ダッシュボードのグラフ表示

- **SimpleMDE** (v1.11以降) または **EasyMDE** (v2.18以降)
  - Markdownエディタ
  - プレビュー機能
  - ツールバー

### 3.3 CSS
- **カスタムCSS** (site.css, admin.css)
  - Bootstrapのカスタマイズ
  - 独自スタイル定義

## 4. 開発支援ツール・ライブラリ

### 4.1 パッケージ管理
- **NuGet** - .NETパッケージ管理
- **npm** - フロントエンドパッケージ管理（必要に応じて）

### 4.2 テスト
- **xUnit** (v2.6以降)
  - 単体テストフレームワーク
  
- **Moq** (v4.20以降)
  - モッキングライブラリ

- **FluentAssertions** (v6.12以降)
  - アサーションライブラリ

### 4.3 コード品質
- **StyleCop.Analyzers**
  - コード規約チェック

- **SonarAnalyzer.CSharp**
  - コード品質分析

## 5. NuGetパッケージ一覧

### 必須パッケージ
```xml
<!-- 既にインストール済み -->
<PackageReference Include="Microsoft.AspNetCore.Mvc" Version="9.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="9.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="9.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="9.0.0" />

<!-- 追加予定 -->
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="9.0.0" />
<PackageReference Include="Markdig" Version="0.37.0" />
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
<PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.1" />
<PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />
```

### 開発・テスト用パッケージ
```xml
<PackageReference Include="xUnit" Version="2.6.1" />
<PackageReference Include="Moq" Version="4.20.69" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="9.0.0" />
```

## 6. デプロイ・ホスティング

### 6.1 想定環境
- **Azure App Service** - PaaS
- **IIS (Internet Information Services)** - オンプレミス
- **Docker + Kubernetes** - コンテナ化（将来的に）

### 6.2 CI/CD
- **GitHub Actions** - 自動ビルド・デプロイ
- **Azure DevOps** - 代替案

## 7. その他のツール・サービス

### 7.1 画像管理
- **ローカルファイルシステム** (wwwroot/images/)
  - シンプルな実装
  
- **Azure Blob Storage** (将来的に)
  - スケーラブルなストレージ

### 7.2 メール送信（コメント通知など）
- **SMTP** - 基本的なメール送信
- **SendGrid** - クラウドメールサービス（将来的に）

## 8. ブラウザサポート

### 対応ブラウザ
- Google Chrome（最新2バージョン）
- Microsoft Edge（最新2バージョン）
- Firefox（最新2バージョン）
- Safari（最新2バージョン）

### モバイル対応
- iOS Safari
- Android Chrome

## 9. 開発フェーズ別技術導入計画

### フェーズ1: 基本機能（MVP）
- Entity Framework Core
- ASP.NET Core Identity
- Markdig
- Bootstrap 5

### フェーズ2: 機能拡張
- AutoMapper
- FluentValidation
- Serilog
- SimpleMDE/EasyMDE

### フェーズ3: 最適化
- キャッシュ機能
- Chart.js（ダッシュボード）
- Azure Blob Storage（画像）

### フェーズ4: テスト・品質向上
- xUnit
- Moq
- FluentAssertions
- コード分析ツール
