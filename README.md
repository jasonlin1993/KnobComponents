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
- 沒辦法用滑鼠拖曳元件
- 外圍環形拉條顯示錯誤

#### 1-2 如何修正?
