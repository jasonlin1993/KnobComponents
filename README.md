## 背景模式面試題目 - Knob 元件製作

### 元件功能需求
- 拉動元件必須可以變更數值
- 思考並設計元件 Emmit Events
- 中間顯示目前數值，外圍有環形拉條。
- 可自訂最大、最小值
- 可自訂顏色與尺寸
- 可設定起點位置
  
---

### 我的思考過程

#### 1. 首先問 chatgpt 並快速產出第 1 版本的 knob components
https://chatgpt.com/share/6789e4ca-918c-8012-a9fc-e6bdfe828b50

複製貼上後產生一個很像，但完全不對的 Knob 元件，可是我們有第 1 個版本了。
![image](https://github.com/jasonlin1993/KnobComponents/blob/main/knob.gif)

#### 1-1 哪裡需要修正?
 1. 拖曳 vs 滑鼠移動：滑鼠只要進入範圍就開始計算，而不是在「按住並拖曳」時才更新數值。
 2. 外圍環形拉條比例顯示錯誤：如果想讓旋轉對應到 0~100 的百分比，需要對角度和進度值做更多細部校正

#### 1-2 程式碼如何修正?
在改進版本中，重點是解決以下問題：
1. 拖曳啟動與停止
   - 只在 `mousedown` / touchstart 時開始偵測移動，並在 mouseup / touchend 時停止偵測。
