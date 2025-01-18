<template>
  <div class="flex h-screen w-full flex-col items-center justify-center">
    <div
      class="relative h-64 w-64 select-none"
      @mousedown="startDrag"
      @touchstart="startDrag"
      ref="container"
    >
      <svg class="h-full w-full" viewBox="0 0 100 100">
        <!-- 底層圓圈 -->
        <circle cx="50" cy="50" r="45" fill="none" stroke="#e2e8f0" stroke-width="8" />
        <!-- 進度圓圈 -->
        <circle
          cx="50"
          cy="50"
          r="45"
          fill="none"
          stroke="#3b82f6"
          stroke-width="8"
          stroke-linecap="round"
          :stroke-dasharray="circumference"
          :stroke-dashoffset="dashOffset"
          transform="rotate(-90 50 50)"
        />
        <!-- 可拖曳的圓點 -->
        <circle
          :cx="handlePosition.x"
          :cy="handlePosition.y"
          r="6"
          fill="#2563eb"
          class="cursor-grab active:cursor-grabbing"
          @mousedown.stop="startDrag"
          @touchstart.stop="startDrag"
        />
      </svg>

      <!-- 中間顯示的文字 (現在顯示的值) -->
      <!-- 加上 @click 事件，點擊後打開彈窗 -->
      <div
        class="absolute inset-0 flex items-center justify-center text-4xl font-bold text-gray-800"
        aria-live="polite"
        @click="openRangePopup"
      >
        {{ Math.round(displayValue) }}
      </div>
    </div>

    <div
      v-if="showRangePopup"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-40"
    >
      <div class="w-64 rounded bg-white p-4 shadow-lg">
        <h2 class="mb-4 text-xl font-bold">設定最小值 / 最大值</h2>
        <div class="mb-2">
          <label class="mb-1 block">最小值:</label>
          <input type="number" v-model.number="minValue" class="w-full rounded border px-2 py-1" />
        </div>
        <div class="mb-2">
          <label class="mb-1 block">最大值:</label>
          <input type="number" v-model.number="maxValue" class="w-full rounded border px-2 py-1" />
        </div>
        <div class="text-right">
          <button class="rounded bg-blue-500 px-4 py-1 text-white" @click="showRangePopup = false">
            確定
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted } from 'vue';

export default defineComponent({
  name: 'EnhancedRotatingProgress',
  setup() {
    // ----- 新增 minValue / maxValue 狀態 -----
    const minValue = ref(0);
    const maxValue = ref(100);

    // 原本的 progress (0~100 表示百分比)
    const progress = ref(0);

    // 顯示/關閉彈窗
    const showRangePopup = ref(false);

    // 彈窗開啟函式
    const openRangePopup = () => {
      showRangePopup.value = true;
    };

    // 計算用的參數
    const container = ref<HTMLElement | null>(null);
    const isDragging = ref(false);
    const startAngle = ref(0);
    const radius = 45;

    // 圓周長
    const circumference = computed(() => 2 * Math.PI * radius);

    // 根據 progress 決定 stroke 的 dashOffset
    const dashOffset = computed(() => {
      return circumference.value - (progress.value / 100) * circumference.value;
    });

    // 進度圓點 (handle) 的座標
    const handlePosition = computed(() => {
      const angle = (progress.value / 100) * 2 * Math.PI - Math.PI / 2;
      return {
        x: 50 + 45 * Math.cos(angle),
        y: 50 + 45 * Math.sin(angle),
      };
    });

    // 將 progress(0~100) 對應到 [minValue, maxValue] 後的真正顯示數值
    const displayValue = computed(() => {
      // 先把 progress 當成 0~1 的比例，再映射到 minValue~maxValue
      const ratio = progress.value / 100;
      return minValue.value + (maxValue.value - minValue.value) * ratio;
    });

    // 計算滑鼠當前角度
    const calculateAngle = (event: MouseEvent | TouchEvent) => {
      if (!container.value) return 0;

      const rect = container.value.getBoundingClientRect();
      const centerX = rect.left + rect.width / 2;
      const centerY = rect.top + rect.height / 2;

      const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
      const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

      const deltaX = clientX - centerX;
      const deltaY = clientY - centerY;

      let angle = Math.atan2(deltaY, deltaX);
      angle = angle * (180 / Math.PI);
      if (angle < 0) angle += 360;

      return angle;
    };

    // 拖曳時更新 progress
    const updateProgress = (event: MouseEvent | TouchEvent) => {
      if (!isDragging.value) return;

      const currentAngle = calculateAngle(event);
      let angleDiff = currentAngle - startAngle.value;

      if (angleDiff > 180) angleDiff -= 360;
      if (angleDiff < -180) angleDiff += 360;

      let newProgress = progress.value + (angleDiff / 360) * 100;

      // 讓數值始終在 0~100 間
      if (newProgress < 0) {
        newProgress += 100;
      }
      if (newProgress > 100) {
        newProgress -= 100;
      }

      progress.value = newProgress;
      startAngle.value = currentAngle;
    };

    // 開始拖曳
    const startDrag = (event: MouseEvent | TouchEvent) => {
      event.preventDefault();
      isDragging.value = true;
      startAngle.value = calculateAngle(event);

      document.addEventListener('mousemove', updateProgress);
      document.addEventListener('touchmove', updateProgress);
      document.addEventListener('mouseup', stopDrag);
      document.addEventListener('touchend', stopDrag);
    };

    // 停止拖曳
    const stopDrag = () => {
      isDragging.value = false;
      document.removeEventListener('mousemove', updateProgress);
      document.removeEventListener('touchmove', updateProgress);
      document.removeEventListener('mouseup', stopDrag);
      document.removeEventListener('touchend', stopDrag);
    };

    onMounted(() => {
      document.addEventListener('mouseleave', stopDrag);
    });

    onUnmounted(() => {
      document.removeEventListener('mouseleave', stopDrag);
    });

    return {
      // 原本的
      progress,
      container,
      circumference,
      dashOffset,
      handlePosition,
      startDrag,

      // 新增的
      minValue,
      maxValue,
      showRangePopup,
      openRangePopup,
      displayValue,
    };
  },
});
</script>
