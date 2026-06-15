# 修正地球上看到的月相圖形為圓形

將「軌道模擬器」中「地球上看到的月相」由目前的垂直橢圓形（變形）修正為完美的正圓形，並提升其視覺美學，使其與主題的暗色太空風格一致。

## 問題原因分析

1. **Canvas 繪圖比例拉伸（主要原因）**：
   在 [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) 中，地球月相的 `<canvas id="moonPhasePreview">` 沒有在 HTML 屬性中指定 `width` 與 `height`。這導致瀏覽器預設其畫布大小為 `300x150`。然而，CSS 樣式中對 `.preview-canvas` 設定了 `width: 200px; height: 200px;` 的正方形尺寸。這使得預設的 `300x150` 圖像被強行擠壓拉伸至 `200x200`，造成原本應為圓形的月相在畫面上顯示為垂直方向拉長的橢圓形。
   
2. **視覺背景不協調**：
   月相預覽區域 `#moonPreview` 的背景色為亮灰色（`#f8f9fa`），與整體網頁的暗藍色太空主題不搭，且 Canvas 呈現的星空背景是矩形的，不夠精緻。

---

## 提案修改內容

### 1. 修正 Canvas 解析度拉伸
在 HTML 中為 `<canvas id="moonPhasePreview">` 加上明確的 `width="200" height="200"` 屬性，使其內部繪圖解析度與 CSS 顯示大小一致，消除橢圓變形。

### 2. 套用圓形遮罩與陰影
在 CSS 中為 `.preview-canvas` 加上 `border-radius: 50%` 與 `box-shadow`。這能將星空背景也裁切為正圓形，並加上外發光效果，呈現出如同透過天文望遠鏡觀測月球的質感。

### 3. 優化預覽容器背景色
將 `#moonPreview` 的背景色從亮灰色（`#f8f9fa`）改為半透明深藍色（`rgba(10, 20, 40, 0.4)`），邊框改為與系統一致的淡藍色，使其與網頁暗色風格完美契合。

---

## Proposed Changes

### 軌道模擬器 UI 與樣式調整

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html)

**CSS 部分 (約 L155 - L169)**
```diff
         #moonPreview {
             width: 100%;
             height: 250px;
-            border: 2px solid #667eea;
-            border-radius: 10px;
-            background: #f8f9fa;
+            border: 2px solid rgba(65, 105, 225, 0.3);
+            border-radius: 10px;
+            background: rgba(10, 20, 40, 0.4);
             display: flex;
             align-items: center;
             justify-content: center;
         }
 
         .preview-canvas {
             width: 200px;
             height: 200px;
+            border-radius: 50%;
+            overflow: hidden;
+            box-shadow: 0 0 20px rgba(65, 105, 225, 0.5);
         }
```

**HTML 部分 (約 L662 - L664)**
```diff
                     <div class="preview-title">地球上看到的月相</div>
                     <div id="moonPreview">
-                        <canvas id="moonPhasePreview" class="preview-canvas"></canvas>
+                        <canvas id="moonPhasePreview" class="preview-canvas" width="200" height="200"></canvas>
                     </div>
```

---

## Verification Plan

### Manual Verification
1. 用瀏覽器開啟 `index.html`。
2. 觀察「地球上看到的月相」圖形，確認其是否為完美的圓形（不再是橢圓形）。
3. 拖曳月球軌道上的月球，觀察月相的亮暗變化是否正常運作，確認 Canvas 繪製功能未受影響。
4. 檢查背景配色是否融入系統暗色主題，且圓形外框是否有好看的發光效果。
