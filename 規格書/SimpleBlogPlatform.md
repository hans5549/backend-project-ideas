# Simple Blog Platform API 規格

## 專案概述
這是一個簡單的部落格平台 API，提供文章管理、評論系統和互動功能。主要目標是建立一個輕量級但功能完整的部落格系統，讓使用者可以方便地發布文章、管理內容並與讀者互動。

## 技術棧
- 後端框架：ASP.NET Core Web API
- 資料庫：MySQL 8.0
- ORM：Entity Framework Core
- 認證：JWT
- API 文檔：Swagger
- 快取：Redis

## 核心功能

### 1. 文章管理 (Posts)

#### 文章基本資訊
- 標題 (title) - 文章標題，限制 100 字元
- 內容 (content) - 支援 Markdown 格式，無字數限制
- 摘要 (summary) - 自動從內容生成，限制 200 字元
- 狀態 (status) - published/draft，預設為 draft
- 建立時間 (created_at) - 自動記錄
- 更新時間 (updated_at) - 自動更新
- 瀏覽次數 (view_count) - 預設為 0
- 按讚數 (like_count) - 預設為 0
- 收藏數 (favorite_count) - 預設為 0

#### 文章列表功能
- 分頁 (page, page_size) - 預設每頁 10 筆
- 排序 (最新/最熱門/最多瀏覽)
- 狀態過濾 (已發布/草稿)
- 標籤過濾
- 時間範圍過濾
- 熱門文章推薦

#### 文章搜尋
- 標題搜尋
- 內容搜尋
- 標籤搜尋
- 模糊匹配
- 搜尋結果高亮
- 搜尋歷史記錄
- 相關文章推薦

#### 標籤系統
- 標籤 CRUD
- 文章標籤關聯
- 熱門標籤統計
- 標籤雲功能
- 標籤自動建議

### 2. 評論系統 (Comments)

#### 評論基本資訊
- 內容 (content) - 限制 1000 字元
- 建立時間 (created_at) - 自動記錄
- 更新時間 (updated_at) - 自動更新
- 父評論 ID (parent_id) - 用於回覆功能
- 評論深度 (depth) - 用於巢狀評論，限制最多 3 層
- 排序順序 (order) - 用於同層評論排序
- 狀態 (status) - active/hidden，預設為 active

#### 評論功能
- 新增評論
- 刪除評論
- 評論列表 (分頁)
- 評論回覆
- 評論排序 (最新/最熱門)
- 評論過濾 (時間/狀態)
- 評論通知
- 評論舉報功能

### 3. 互動功能

#### 按讚系統
- 按讚/取消按讚
- 按讚數統計
- 按讚列表
- 防止重複按讚
- 按讚通知
- 按讚趨勢分析

#### 收藏系統
- 收藏/取消收藏
- 收藏數統計
- 收藏列表
- 防止重複收藏
- 收藏分類
- 收藏匯出功能

## 資料庫設計

### 資料表結構

#### Posts 表
```sql
CREATE TABLE `posts` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '文章ID',
    `title` varchar(100) NOT NULL COMMENT '文章標題',
    `content` text NOT NULL COMMENT '文章內容',
    `summary` varchar(200) DEFAULT NULL COMMENT '文章摘要',
    `status` varchar(20) NOT NULL DEFAULT 'draft' COMMENT '文章狀態',
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
    `view_count` int(11) NOT NULL DEFAULT '0' COMMENT '瀏覽次數',
    `like_count` int(11) NOT NULL DEFAULT '0' COMMENT '按讚數',
    `favorite_count` int(11) NOT NULL DEFAULT '0' COMMENT '收藏數',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='文章表';
```

#### Comments 表
```sql
CREATE TABLE `comments` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '評論ID',
    `post_id` bigint(20) NOT NULL COMMENT '文章ID',
    `content` varchar(1000) NOT NULL COMMENT '評論內容',
    `parent_id` bigint(20) DEFAULT NULL COMMENT '父評論ID',
    `depth` int(11) NOT NULL DEFAULT '0' COMMENT '評論深度',
    `order` int(11) NOT NULL DEFAULT '0' COMMENT '排序順序',
    `status` varchar(20) NOT NULL DEFAULT 'active' COMMENT '評論狀態',
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='評論表';
```

#### Tags 表
```sql
CREATE TABLE `tags` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '標籤ID',
    `name` varchar(50) NOT NULL COMMENT '標籤名稱',
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='標籤表';
```

#### Post_Tags 表
```sql
CREATE TABLE `post_tags` (
    `post_id` bigint(20) NOT NULL COMMENT '文章ID',
    `tag_id` bigint(20) NOT NULL COMMENT '標籤ID',
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    PRIMARY KEY (`post_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='文章標籤關聯表';
```

#### Likes 表
```sql
CREATE TABLE `likes` (
    `post_id` bigint(20) NOT NULL COMMENT '文章ID',
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    PRIMARY KEY (`post_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='按讚表';
```

#### Favorites 表
```sql
CREATE TABLE `favorites` (
    `post_id` bigint(20) NOT NULL COMMENT '文章ID',
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    PRIMARY KEY (`post_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='收藏表';
```

## API 端點

### 文章相關
- GET /api/posts - 取得文章列表
- GET /api/posts/{id} - 取得單篇文章
- POST /api/posts - 新增文章
- PUT /api/posts/{id} - 更新文章
- DELETE /api/posts/{id} - 刪除文章
- GET /api/posts/search - 搜尋文章
- GET /api/posts/tags/{tag} - 取得特定標籤的文章
- GET /api/posts/popular - 取得熱門文章
- GET /api/posts/related/{id} - 取得相關文章

### 評論相關
- GET /api/posts/{id}/comments - 取得文章評論
- POST /api/posts/{id}/comments - 新增評論
- DELETE /api/comments/{id} - 刪除評論
- PUT /api/comments/{id}/report - 舉報評論
- GET /api/comments/recent - 取得最新評論

### 互動相關
- POST /api/posts/{id}/like - 按讚文章
- POST /api/posts/{id}/favorite - 收藏文章
- GET /api/posts/{id}/likes - 取得按讚數
- GET /api/posts/{id}/favorites - 取得收藏數
- GET /api/posts/{id}/trends - 取得互動趨勢

### 標籤相關
- GET /api/tags - 取得所有標籤
- GET /api/tags/{id}/posts - 取得標籤相關文章
- GET /api/tags/popular - 取得熱門標籤
- GET /api/tags/suggestions - 取得標籤建議

## API 響應格式

### 成功響應
```json
{
  "success": true,
  "data": {
    // 實際數據
  },
  "meta": {
    "page": 1,
    "page_size": 10,
    "total": 100,
    "total_pages": 10,
    "has_next": true,
    "has_previous": false
  }
}
```

### 錯誤響應
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "錯誤訊息",
    "details": {
      "field": "錯誤欄位",
      "reason": "錯誤原因"
    }
  }
}
```

## 效能優化

### 1. 資料庫查詢優化
- 適當的索引設計
- 避免 N+1 問題
- 使用 JOIN 優化查詢
- 定期資料庫維護
- 查詢效能監控

### 2. 快取策略
- 熱門文章快取
- 標籤列表快取
- 評論列表快取
- 使用者資料快取
- 快取更新策略

### 3. 分頁優化
- 使用 cursor-based 分頁
- 限制每頁數量
- 避免深分頁問題
- 預加載機制
- 無限滾動支援

## 實作順序建議
1. 文章 CRUD 基礎功能
2. 文章列表與分頁
3. 標籤系統
4. 評論功能
5. 按讚/收藏功能
6. 搜尋功能
7. 效能優化
8. 監控與日誌
9. 部署與維護

## 注意事項
1. 所有 API 都需要進行輸入驗證
2. 實作適當的錯誤處理機制
3. 確保資料庫操作的原子性
4. 實作適當的日誌記錄
5. 考慮資料備份策略
6. 實作適當的安全措施
7. 考慮系統擴展性 