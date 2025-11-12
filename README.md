# Dify Helper Tool

一個強大的 Dify 工作流程輔助工具，提供節點匯入匯出與專案類型轉換功能。

## 功能特點

### 1. 節點匯入匯出
- 從來源 DSL 檔案選擇性匯入節點到目標專案
- 視覺化圖形介面，直觀展示節點關係
- 支援多種選擇方式：
  - 單個節點/連接點擊選擇
  - 框選批量選擇
  - 列表勾選
  - 全選/全不選功能
- 自動處理節點 ID 衝突，避免覆蓋現有節點
- 支援跨類型匯入（Chatflow → Workflow 自動轉換）

### 2. 專案類型轉換
- **Chatflow → Workflow**
  - `answer` 節點自動轉換為 `end` 節點
  - `sys.query` 轉換為 `query`
  - 自動清理 `sys.dialogue_count`、`sys.conversation_id` 等 Chatflow 專用變數
- **Workflow → Chatflow**
  - `end` 節點自動轉換為 `answer` 節點
  - `query` 轉換為 `sys.query`
  - 自動處理節點輸出格式差異

### 3. 視覺化圖形展示
- 基於 Cytoscape.js 的互動式流程圖
- Dagre 佈局算法，自動排列節點
- 支援縮放、平移、框選等操作
- 即時視覺化回饋選擇狀態

## 技術棧

- **前端框架**: 純 HTML/CSS/JavaScript（無需構建工具）
- **YAML 解析**: js-yaml 4.1.0
- **圖形渲染**: Cytoscape 3.26.0
- **佈局引擎**: Dagre 0.8.5 + cytoscape-dagre 2.5.0
- **樣式**: 原生 CSS3（漸層、動畫、響應式設計）

## 使用說明

### 環境需求
- 現代瀏覽器（Chrome、Firefox、Safari、Edge）
- 網路連接（用於載入 CDN 資源）

### 快速開始

1. **直接開啟檔案**
   ```bash
   # 直接用瀏覽器開啟 index.html
   open index.html  # macOS
   start index.html # Windows
   ```

2. **使用本地伺服器（推薦）**
   ```bash
   # Python 3
   python -m http.server 8000

   # Node.js
   npx serve

   # 然後瀏覽器訪問 http://localhost:8000
   ```

### 節點匯入匯出操作流程

1. **上傳檔案**
   - 點擊「來源檔案」區域，選擇包含要匯出節點的 DSL 檔案
   - 點擊「目標檔案」區域，選擇要被匯入的目標專案 DSL 檔案

2. **選擇節點**
   - 系統會自動顯示視覺化流程圖和節點列表
   - 預設全選所有節點（不含 start 節點）
   - 可透過以下方式調整選擇：
     - **圖形操作**: 左鍵點擊切換、左鍵拖曳框選、右鍵拖動移動畫布、滾輪縮放
     - **列表操作**: 勾選或點擊節點項目
     - **批量操作**: 使用全選/全不選按鈕

3. **執行匯入**
   - 確認選擇後，點擊「執行匯入」按鈕
   - 系統會自動下載合併後的 DSL 檔案

### 專案類型轉換操作流程

1. **上傳檔案**
   - 點擊「上傳 DSL 檔案」選擇 Chatflow 或 Workflow 檔案

2. **查看分析結果**
   - 系統自動偵測專案類型
   - 顯示節點和連接數量統計
   - 顯示當前類型和目標類型

3. **執行轉換**
   - 點擊「轉換類型」按鈕執行轉換
   - 轉換成功後可下載新檔案

## 核心功能說明

### 自動類型轉換

當從 Chatflow 匯入節點到 Workflow 時，工具會自動處理：

```yaml
# Chatflow answer 節點
data:
  type: answer
  answer: "{{#llm.text#}}"
  variables: []

# 自動轉換為 Workflow end 節點
data:
  type: end
  outputs:
    - value_selector: ["llm", "text"]
      value_type: string
      variable: text
```

### ID 衝突處理

當匯入的節點 ID 與目標專案衝突時，系統會自動生成新的 ID：

```javascript
// 原始 ID: "1234567890"
// 衝突後自動重新命名: "1234567890_1699999999999_abc1234"
```

### 視覺化操作指南

| 操作 | 說明 |
|------|------|
| 左鍵點擊節點/邊 | 切換選中狀態 |
| 左鍵拖曳空白處 | 框選多個元素 |
| 右鍵拖曳 | 移動畫布 |
| 滾輪 | 縮放視圖 |
| 重置視圖按鈕 | 自動調整至最佳視角 |

## 注意事項

1. **DSL 檔案格式**
   - 僅支援 `.yml` 或 `.yaml` 格式
   - 必須是有效的 Dify DSL 結構

2. **節點相容性**
   - `start` 節點會被自動過濾，不會被匯入
   - 跨類型匯入時，僅 `answer`/`end` 節點會被轉換
   - 其他節點類型保持原樣

3. **資料安全**
   - 所有處理完全在瀏覽器本地執行
   - 不會上傳任何資料到伺服器
   - 原始檔案不會被修改

4. **瀏覽器相容性**
   - 建議使用最新版本的現代瀏覽器
   - 需要啟用 JavaScript

## 檔案結構

```
dify_tool/
├── index.html          # 主應用程式（包含所有功能）
├── README.md          # 本說明文件
└── .gitignore         # Git 忽略規則
```

## 開發說明

### 核心函數

- `renderGraph()`: 渲染 Cytoscape 視覺化圖形
- `executeImport()`: 執行節點匯入邏輯
- `convertType()`: 執行專案類型轉換
- `parseVariableSelector()`: 解析變數選擇器格式
- `convertVariableReferences()`: 轉換變數引用

### 自訂修改

所有樣式和邏輯都在 `index.html` 中，可以直接修改：

- **樣式**: 修改 `<style>` 標籤內的 CSS
- **邏輯**: 修改 `<script>` 標籤內的 JavaScript
- **佈局**: 修改 Cytoscape 的 `layout` 配置

## 常見問題

**Q: 匯入後節點位置重疊怎麼辦？**
A: 在 Dify 編輯器中手動調整節點位置，或使用編輯器的自動排列功能。

**Q: 轉換後的專案能否直接使用？**
A: 大部分情況可以直接使用，但建議在 Dify 中測試確認，特別是涉及複雜變數引用的場景。

**Q: 為什麼 start 節點不會被匯入？**
A: start 節點是每個專案的入口點，應該保持唯一性，因此被自動過濾。

**Q: 支援批量處理多個檔案嗎？**
A: 目前僅支援單個檔案處理，批量處理功能計劃在未來版本中加入。

## 更新日誌

### v1.0.0 (2024)
- 初始版本發布
- 節點匯入匯出功能
- 專案類型轉換功能
- 視覺化圖形展示
- 自動 ID 衝突處理

## 授權

本專案使用 MIT 授權條款。

## 致謝

- [Dify](https://dify.ai/) - 強大的 LLM 應用開發平台
- [Cytoscape.js](https://js.cytoscape.org/) - 圖形視覺化庫
- [js-yaml](https://github.com/nodeca/js-yaml) - YAML 解析器

---

Made with ❤️ for Dify Users
