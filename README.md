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
[https://chatgpt.com/share/6789e4ca-918c-8012-a9fc-e6bdfe828b50](https://chatgpt.com/share/6789e4ca-918c-8012-a9fc-e6bdfe828b50)

複製貼上後產生一個很像，但完全不對的 Knob 元件，可是我們有第 1 個版本了。  
![image](https://github.com/jasonlin1993/KnobComponents/blob/main/knob.gif)

#### 1-1 哪裡需要修正?
1. **拖曳與滑鼠移動的差異**
   第一版是用 `@mousemove` 直接監聽滑鼠在整個圓形範圍的移動狀況，導致只要滑進圓形區域就會改變進度。
2. **環形進度（stroke-dashoffset）的計算與角度對應問題**
   第一版是用相對於整個 `div` 寬高的滑鼠座標，去計算角度並得出進度，但沒有做任何「拖曳啟動（drag start）」或「拖曳停止（drag stop）」等事件管理。

#### 1-2 程式碼如何修正?
為了讓 Knob 元件在「按住並拖曳」時更新數值，需要 **啟動拖曳模式** 、**計算滑鼠相對角度** 並將其映射到 0~100 的進度區間。

```ts
// 1. 按下後才開始監聽 mousemove/touchmove 事件
const startDrag = (event: MouseEvent | TouchEvent) => {
  event.preventDefault();
  isDragging.value = true;
  startAngle.value = calculateAngle(event);
  document.addEventListener('mousemove', updateProgress);
  document.addEventListener('touchmove', updateProgress);
  document.addEventListener('mouseup', stopDrag);
  document.addEventListener('touchend', stopDrag);
};

// 2. 拖曳過程中更新進度
const updateProgress = (event: MouseEvent | TouchEvent) => {
  if (!isDragging.value) return;
  const currentAngle = calculateAngle(event);
  // ... 根據 currentAngle 與 startAngle 做角度差計算，更新 progress
  // ... 並確保進度在 0~100 之間循環
  startAngle.value = currentAngle;
};

// 3. 放開滑鼠/手指時，停止監聽並結束拖曳
const stopDrag = () => {
  isDragging.value = false;
  document.removeEventListener('mousemove', updateProgress);
  document.removeEventListener('touchmove', updateProgress);
  document.removeEventListener('mouseup', stopDrag);
  document.removeEventListener('touchend', stopDrag);
};
```

#### 現在有一個最基礎的功能版本:拉動元件必須可以變更數值
![image](https://github.com/jasonlin1993/KnobComponents/blob/main/knob1.gif)

---

#### 2. 新增元件功能: 可自訂最大、最小值




