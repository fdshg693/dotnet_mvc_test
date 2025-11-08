# ブログシステム 実装計画

## 1. 開発スケジュール概要

### 全体工程
```
フェーズ1: データベース設計・構築     （1週間）
フェーズ2: 認証機能実装               （3日）
フェーズ3: 記事管理機能実装           （1週間）
フェーズ4: カテゴリ・タグ機能実装     （3日）
フェーズ5: コメント機能実装           （3日）
フェーズ6: 検索機能実装               （2日）
フェーズ7: 管理画面実装               （5日）
フェーズ8: フロントエンド実装         （5日）
フェーズ9: テスト・バグ修正           （5日）
フェーズ10: デプロイ・リリース        （2日）
```

## 2. フェーズ別実装計画

### フェーズ1: データベース設計・構築

#### タスク
1. **Entity クラスの作成**
   - Article.cs
   - Category.cs
   - Tag.cs
   - Comment.cs
   - ApplicationUser.cs (Identity拡張)

2. **DbContext の作成**
   - ApplicationDbContext.cs
   - エンティティの設定（Fluent API）
   - リレーションの定義

3. **マイグレーションの実行**
   - Initial Migration 作成
   - データベース作成
   - 初期データのシード

#### 成果物
- Models/Entities/配下のクラスファイル
- Data/ApplicationDbContext.cs
- マイグレーションファイル

---

### フェーズ2: 認証機能実装

#### タスク
1. **ASP.NET Core Identity の設定**
   - Identity の追加
   - ApplicationUser の設定
   - ロール定義（Administrator, User）

2. **ログイン・ログアウト機能**
   - AccountController 作成
   - Login.cshtml 作成
   - 認証Cookie設定

3. **ユーザー管理**
   - 初期管理者ユーザーの作成
   - ユーザー登録機能（オプション）

#### 成果物
- Controllers/AccountController.cs
- Views/Account/Login.cshtml
- 認証設定（Program.cs）

---

### フェーズ3: 記事管理機能実装

#### タスク
1. **Service層の実装**
   - IArticleService インターフェース
   - ArticleService クラス
   - ビジネスロジックの実装

2. **Controller の実装**
   - Admin/ArticleController（管理側）
   - CRUD操作メソッド

3. **ViewModel の作成**
   - ArticleCreateViewModel
   - ArticleEditViewModel
   - ArticleListViewModel
   - ArticleDetailViewModel

4. **View の作成**
   - 記事一覧画面（管理）
   - 記事作成画面
   - 記事編集画面

5. **Markdown対応**
   - Markdig統合
   - プレビュー機能

#### 成果物
- Services/IArticleService.cs, ArticleService.cs
- Controllers/Admin/ArticleController.cs
- Models/ViewModels/Admin/Article*.cs
- Views/Admin/Article/*.cshtml

---

### フェーズ4: カテゴリ・タグ機能実装

#### タスク
1. **Service層の実装**
   - ICategoryService, ITagService
   - CategoryService, TagService

2. **Controller の実装**
   - Admin/CategoryController
   - Admin/TagController

3. **ViewModel の作成**
   - Category, Tag用のViewModel

4. **View の作成**
   - カテゴリ管理画面
   - タグ管理画面

5. **記事とのリレーション処理**
   - カテゴリ選択UI
   - タグ入力UI（複数選択）

#### 成果物
- Services/ICategoryService.cs, CategoryService.cs
- Services/ITagService.cs, TagService.cs
- Controllers/Admin/CategoryController.cs, TagController.cs
- Views/Admin/Category/*.cshtml, Tag/*.cshtml

---

### フェーズ5: コメント機能実装

#### タスク
1. **Service層の実装**
   - ICommentService
   - CommentService

2. **Controller の実装**
   - CommentController（投稿用）
   - Admin/CommentController（管理用）

3. **ViewModel の作成**
   - CommentCreateViewModel
   - CommentListViewModel

4. **View の作成**
   - コメント投稿フォーム（記事詳細ページに埋め込み）
   - コメント管理画面

5. **承認機能の実装**
   - 承認/非承認切り替え
   - 未承認コメントの表示制御

#### 成果物
- Services/ICommentService.cs, CommentService.cs
- Controllers/CommentController.cs
- Controllers/Admin/CommentController.cs
- Views/Admin/Comment/*.cshtml

---

### フェーズ6: 検索機能実装

#### タスク
1. **検索Service の実装**
   - ArticleServiceに検索メソッド追加
   - LINQ による全文検索

2. **Controller の実装**
   - BlogController に Search アクション

3. **View の作成**
   - 検索フォーム（共通レイアウトに配置）
   - 検索結果ページ

#### 成果物
- 検索メソッド追加（ArticleService）
- Views/Blog/Search.cshtml

---

### フェーズ7: 管理画面実装

#### タスク
1. **管理画面レイアウトの作成**
   - _AdminLayout.cshtml
   - 管理画面用CSS (admin.css)
   - サイドバーナビゲーション

2. **ダッシュボードの実装**
   - DashboardController
   - 統計情報表示
   - グラフ表示（Chart.js）

3. **共通機能の実装**
   - ページング
   - ソート機能
   - フィルター機能

#### 成果物
- Views/Shared/_AdminLayout.cshtml
- Controllers/Admin/DashboardController.cs
- Views/Admin/Dashboard/Index.cshtml
- wwwroot/css/admin.css

---

### フェーズ8: フロントエンド実装

#### タスク
1. **BlogController の実装**
   - 記事一覧（Index）
   - 記事詳細（Detail）
   - カテゴリ別一覧（Category）
   - タグ別一覧（Tag）

2. **View の作成**
   - トップページ（記事一覧）
   - 記事詳細ページ
   - カテゴリ別ページ
   - タグ別ページ

3. **デザインの調整**
   - レスポンシブ対応
   - カードレイアウト
   - タグクラウド

4. **ページング実装**
   - ページャーコンポーネント

#### 成果物
- Controllers/BlogController.cs
- Views/Blog/*.cshtml
- wwwroot/css/site.css（スタイル追加）

---

### フェーズ9: テスト・バグ修正

#### タスク
1. **単体テストの作成**
   - Service層のテスト
   - Controller層のテスト

2. **統合テストの実施**
   - 画面遷移テスト
   - CRUD操作テスト

3. **バグ修正**
   - 発見された不具合の修正
   - エラーハンドリングの強化

4. **パフォーマンステスト**
   - ページ読み込み速度確認
   - データベースクエリ最適化

#### 成果物
- テストプロジェクト
- バグ修正リスト

---

### フェーズ10: デプロイ・リリース

#### タスク
1. **本番環境設定**
   - appsettings.Production.json
   - 接続文字列の設定
   - 環境変数の設定

2. **デプロイ**
   - Azure App Service へのデプロイ
   - または IIS へのデプロイ

3. **本番環境テスト**
   - 動作確認
   - セキュリティチェック

4. **ドキュメント作成**
   - ユーザーマニュアル
   - 管理者マニュアル

#### 成果物
- デプロイ済みアプリケーション
- ドキュメント

---

## 3. 優先順位付き機能リスト

### 必須機能（MVP: Minimum Viable Product）
1. ✅ 記事のCRUD操作
2. ✅ カテゴリ管理
3. ✅ タグ管理
4. ✅ 管理者認証
5. ✅ 記事一覧・詳細表示（フロント）
6. ✅ Markdown対応

### 重要機能
7. コメント機能
8. 検索機能
9. ダッシュボード
10. ページング

### あると良い機能
11. 記事の下書き機能
12. アイキャッチ画像アップロード
13. 記事プレビュー
14. タグクラウド

### 将来的な拡張機能
15. ユーザー登録機能
16. 記事のいいね機能
17. RSS/Atomフィード
18. OGPタグ対応
19. サイトマップ生成
20. SEO最適化

## 4. 技術的な検討事項

### 4.1 画像アップロード
- **方式**: wwwroot/images/ への保存
- **制限**: 最大5MB、jpg/png/gif のみ
- **実装**: IFormFile を使用

### 4.2 URLスラッグ生成
- 日本語タイトルからのローマ字変換
- 一意性の担保
- SlugHelper ユーティリティクラス作成

### 4.3 セキュリティ対策
- [ValidateAntiForgeryToken] の使用
- [Authorize] 属性による認可
- HTMLエスケープ処理
- SQLインジェクション対策（EF Coreで自動）

### 4.4 パフォーマンス最適化
- 記事一覧でのEager Loading
- メモリキャッシュの活用
- インデックスの適切な設定

## 5. リスク管理

### 技術的リスク
| リスク | 影響 | 対策 |
|--------|------|------|
| Markdown表示の不具合 | 中 | Markdigのテスト強化 |
| パフォーマンス問題 | 中 | キャッシュ実装、クエリ最適化 |
| 画像アップロード容量 | 低 | ファイルサイズ制限実装 |

### スケジュールリスク
| リスク | 影響 | 対策 |
|--------|------|------|
| 機能実装の遅延 | 高 | MVP機能に絞る |
| バグ修正に時間がかかる | 中 | テストを早期から実施 |

## 6. 次のステップ

1. **即時開始**: フェーズ1（データベース設計・構築）
2. **環境準備**: 
   - SQL Server のインストール確認
   - 必要なNuGetパッケージのインストール
3. **リポジトリ設定**: Gitブランチ戦略の決定
