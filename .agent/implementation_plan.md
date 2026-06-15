# 增加觀測者地球自轉與一日時間對觀月角度（傾角）的影響

實作物理模擬，讓學生能直觀理解「因為地球自轉，觀測者在不同時間看同一個月亮時，月相在天空中呈現的傾斜角度與仰角會隨之變化」的天文物理現象。

## 設計方案

### 1. UI 與操作項擴充
在播放控制區下方，新增一個「觀測者一日時間 (地球自轉)」的 Slider 拉桿（範圍 0° 到 360°）：
* **0°**：正午 (12:00) ── 太陽在天頂
* **90°**：黃昏 (18:00) ── 太陽在西方地平線
* **180°**：午夜 (24:00) ── 太陽在地球正後方
* **270°**：清晨 (06:00) ── 太陽在東方地平線
* 拉桿數值改變時，會更新地球自轉角度 `earthRotation`。

### 2. 軌道模擬器視覺化 (地球上的觀測者)
在 `drawOrbitSimulator` 中，於地球的圓周上，根據 `earthRotation` 角度繪製出：
* **觀測者天頂箭頭** (綠色)：代表觀測者頭頂直指天空的方向。
* **觀測者地平線** (切線)：代表觀測者的視野地平範圍（只有在切線以上的月球才是可見的）。

### 3. 月相傾斜角度（Rotation）模擬
在 `drawMoonPhaseHelper` 中引入 `rotAngle` 參數。在繪製月相前，將 Canvas 座標系旋轉 `rotAngle = (earthRotation - moonAngle) * (Math.PI / 180)`。
* 當觀測者頭部方向旋轉時，在視野中看到的月相就會產生對應的**物理傾斜**（例如升起時橫躺、中天時直立）。

### 4. 精確的仰角與方位角計算
基於物理模型重新計算：
* **仰角 (Elevation)**：`90 - |moonAngle - earthRotation|`。若小於 0° 則顯示為「在地平線下 (不可見)」，並在月相預覽上覆蓋一個半透明的地平遮罩。
* **觀測者方位 (Observer Azimuth)**：依據月球與觀測者天頂的夾角，精準計算出是在東方、南方還是西方天空。

---

## Proposed Changes

### 1. UI 元素擴充
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L653 - L657)
在播放速度下拉選單旁加入自轉拉桿：
```html
                            </div>
                        </div>
                        <div class="control-group" style="width: 100%; margin-top: 15px;">
                            <div class="speed-control" style="width: 100%; display: flex; align-items: center; gap: 15px;">
                                <label style="min-width: 110px;">觀測者時間：</label>
                                <input type="range" id="timeSlider" min="0" max="360" value="90" style="flex: 1; accent-color: #ffd700;" oninput="changeObserverTime()">
                                <span id="timeLabel" style="min-width: 90px; color: #ffd700; font-weight: bold; text-align: right;">黃昏 (18:00)</span>
                            </div>
                        </div>
```

### 2. 樣式調整 (新增不可見遮罩)
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L166 - L170)
新增 `.preview-container` 內的地平下警示標籤樣式：
```css
        .preview-canvas {
            width: 200px;
            height: 200px;
            border-radius: 50%;
            overflow: hidden;
            box-shadow: 0 0 20px rgba(65, 105, 225, 0.5);
            position: relative;
        }

        .horizon-mask {
            position: absolute;
            background: rgba(0, 0, 0, 0.7);
            color: #ef9a9a;
            font-weight: bold;
            padding: 8px 15px;
            border-radius: 5px;
            border: 1px solid rgba(244, 67, 54, 0.3);
            display: none;
        }
```

### 3. JavaScript 邏輯擴充
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L780 - L788)
宣告 `earthRotation` 全域變數（預設 90° 亦即黃昏）：
```javascript
        let earthRotation = 90;
```

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L894 - L929)
在 `drawOrbitSimulator` 繪製地球上的綠色天頂箭頭與地平切線。

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L934 - L1021)
重構 `drawMoonPhaseHelper` 與 `drawMoonPhase`，使其支援 `rotAngle` 參數，並動態旋轉畫布。

---

## Verification Plan

### Manual Verification
1. 用瀏覽器開啟 `index.html`。
2. 拖曳「觀測者時間」拉桿，觀察軌道模擬器中，地球上綠色指針與地平線是否隨之旋轉。
3. 確認月球仰角、方位角等數據正確變化，且當仰角小於 0° 時，月相預覽上會顯示「地平線下不可見」字樣。
4. 鎖定月球在 90° (上弦月)，滑動時間拉桿，觀察「地球上看到的月相」是否隨觀測時間不同產生正確的傾斜度變化。
