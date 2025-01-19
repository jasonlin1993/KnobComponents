<template>
  <div class="flex h-screen w-full flex-col items-center justify-center bg-gray-100">
    <div
      class="relative"
      style="width: 200px; height: 200px"
      ref="container"
      @mousedown="startDrag"
      @touchstart="startDrag"
    >
      <svg class="h-full w-full drop-shadow-lg" viewBox="0 0 100 100">
        <!-- 背景圓 (畫滿整圈) -->
        <path :d="backgroundPath" :fill="backgroundColor" stroke="none" />

        <!-- 進度圓餅 -->
        <path :d="progressPath" :fill="progressColor" stroke="none" />

        <!-- 中心圓孔 (當 radius > 0 時顯示) -->
        <circle
          v-if="radius > 0"
          cx="50"
          cy="50"
          :r="radius"
          :fill="circleFillColor"
          stroke="none"
        />

        <!-- 拖曳控制點 -->
        <circle
          :cx="handlePosition.x"
          :cy="handlePosition.y"
          r="4"
          fill="yellow"
          class="cursor-grab active:cursor-grabbing"
          @mousedown.stop="startDrag"
          @touchstart.stop="startDrag"
        />
      </svg>

      <!-- 中心數值顯示 -->
      <div
        class="absolute inset-0 flex items-center justify-center text-2xl font-bold text-gray-800"
        @click="openRangePopup"
      >
        {{ Math.round(displayValue) }}
      </div>
    </div>

    <div class="mt-4 text-center">
      <!-- 可在此改變 0% 起點，示例：-90=上方, 0=右方, 90=下方, 180=左方 -->
      <label class="mr-2">起始角度 (度數)：</label>
      <input
        type="number"
        v-model.number="startOffsetAngleDeg"
        class="w-24 rounded border px-2 py-1 text-center"
      />
      <p class="mt-2 text-sm text-gray-500">
        -90 代表上方、0 代表右方、90 代表下方、180 代表左方...
      </p>
    </div>

    <!-- 彈窗 (設定參數) -->
    <div
      v-if="showRangePopup"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-40"
    >
      <div class="w-80 rounded bg-white p-4 shadow-lg">
        <h2 class="mb-4 text-xl font-bold">設定參數</h2>

        <!-- 最小值 / 最大值 -->
        <div class="mb-2">
          <label class="mb-1 block">最小值:</label>
          <input type="number" v-model.number="minValue" class="w-full rounded border px-2 py-1" />
        </div>
        <div class="mb-2">
          <label class="mb-1 block">最大值:</label>
          <input type="number" v-model.number="maxValue" class="w-full rounded border px-2 py-1" />
        </div>

        <!-- radius - 添加最大值限制 -->
        <div class="mb-2">
          <label class="mb-1 block">圓半徑 (0-50, 0=整個填滿):</label>
          <input
            type="number"
            v-model.number="radius"
            min="0"
            max="50"
            @input="validateRadius"
            class="w-full rounded border px-2 py-1"
          />
          <span class="text-sm text-gray-500">最大值: 50</span>
        </div>

        <!-- 圓底色 (若有需要) -->
        <div class="mb-2">
          <label class="mb-1 block">圓形底色 :</label>
          <input type="color" v-model="circleFillColor" class="w-full rounded border px-2 py-1" />
        </div>

        <!-- 進度色 -->
        <div class="mb-2">
          <label class="mb-1 block">進度色:</label>
          <input type="color" v-model="progressColor" class="w-full rounded border px-2 py-1" />
        </div>

        <div class="mt-4 text-right">
          <button class="rounded bg-blue-500 px-4 py-1 text-white" @click="closeRangePopup">
            確定
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted, watch } from 'vue';

export default defineComponent({
  name: 'PieProgress',
  setup() {
    // 1. 參數
    const minValue = ref(0);
    const maxValue = ref(100);
    const radius = ref(40);

    // 2. 顏色
    const circleFillColor = ref('#ffffff');
    const backgroundColor = ref('#eeeeee');
    const progressColor = ref('#3b82f6');

    // 3. 進度 (0 ~ 100)
    const progress = ref(50);

    // 4. 彈窗
    const showRangePopup = ref(false);
    const openRangePopup = () => (showRangePopup.value = true);
    const closeRangePopup = () => (showRangePopup.value = false);

    // 5. 定義「0% 的起始角度」(單位：度)
    //   -90 => 頂部是 0%
    //    0  => 右邊是 0%
    //   90  => 下方是 0%
    //   180 => 左方是 0%
    const startOffsetAngleDeg = ref(-90);

    // 6. 拖曳相關
    const container = ref<HTMLElement | null>(null);
    const isDragging = ref(false);

    // 驗證並限制 radius 值
    const validateRadius = () => {
      if (radius.value < 0) {
        radius.value = 0;
      } else if (radius.value > 50) {
        radius.value = 50;
      }
    };
    watch(radius, validateRadius);

    // ============== 計算圓餅圖路徑 ==============
    const calculatePiePath = (startA: number, endA: number, r: number) => {
      // 如果角度覆蓋 2π (超過 360 度)，就是滿圓
      if (endA - startA >= Math.PI * 2) {
        return `M 50 50
                  M 50 0
                  A ${r} ${r} 0 1 1 49.99 0
                  Z`;
      }

      const start = {
        x: 50 + r * Math.cos(startA),
        y: 50 + r * Math.sin(startA),
      };
      const end = {
        x: 50 + r * Math.cos(endA),
        y: 50 + r * Math.sin(endA),
      };

      const largeArcFlag = endA - startA <= Math.PI ? 0 : 1;

      return `M 50 50
                L ${start.x} ${start.y}
                A ${r} ${r} 0 ${largeArcFlag} 1 ${end.x} ${end.y}
                Z`;
    };

    /**
     * 1. 將 startOffsetAngleDeg (度) 轉為 radians
     * 2. 0% 對應 startAngle，100% 對應 startAngle + 2π
     */
    const backgroundPath = computed(() => {
      const startAngle = (startOffsetAngleDeg.value * Math.PI) / 180; // deg => rad
      return calculatePiePath(startAngle, startAngle + Math.PI * 2, 50);
    });

    const progressPath = computed(() => {
      const startAngle = (startOffsetAngleDeg.value * Math.PI) / 180;
      const endAngle = startAngle + (progress.value / 100) * Math.PI * 2;

      // 如果超過 100
      if (progress.value >= 100) {
        return calculatePiePath(startAngle, startAngle + Math.PI * 2, 50);
      }
      return calculatePiePath(startAngle, endAngle, 50);
    });

    // knob(黃點) 的位置
    const handlePosition = computed(() => {
      // 依據 startOffsetAngleDeg + progress.value 推算當前絕對角度
      const angle =
        (startOffsetAngleDeg.value * Math.PI) / 180 + (progress.value / 100) * 2 * Math.PI;

      return {
        x: 50 + 50 * Math.cos(angle),
        y: 50 + 50 * Math.sin(angle),
      };
    });

    // 顯示給使用者的數值
    const displayValue = computed(() => {
      const ratio = progress.value / 100;
      return minValue.value + (maxValue.value - minValue.value) * ratio;
    });

    // ============== 拖曳計算 ==============
    // 取得滑鼠 / 手指相對於圓心的角度 (0~360)
    const getCurrentAngleDeg = (event: MouseEvent | TouchEvent) => {
      if (!container.value) return 0;
      const rect = container.value.getBoundingClientRect();
      const centerX = rect.left + rect.width / 2;
      const centerY = rect.top + rect.height / 2;

      let clientX: number;
      let clientY: number;

      if ('touches' in event) {
        clientX = event.touches[0].clientX;
        clientY = event.touches[0].clientY;
      } else {
        clientX = event.clientX;
        clientY = event.clientY;
      }

      const dx = clientX - centerX;
      const dy = clientY - centerY;

      let angleDeg = (Math.atan2(dy, dx) * 180) / Math.PI; // [-180, 180]
      if (angleDeg < 0) {
        angleDeg += 360; // => [0, 360)
      }
      return angleDeg;
    };

    const updateProgress = (event: MouseEvent | TouchEvent) => {
      if (!isDragging.value) return;
      const currentAngleDeg = getCurrentAngleDeg(event);

      // 計算「相對於 startOffsetAngleDeg」的角度 (0~360)
      let relativeDeg = currentAngleDeg - startOffsetAngleDeg.value;
      // 確保在 0~360
      relativeDeg = (relativeDeg + 360) % 360;

      // 換算成 0~100
      let newProgress = (relativeDeg / 360) * 100;
      newProgress = Math.max(0, Math.min(100, newProgress));
      progress.value = newProgress;
    };

    const startDrag = (event: MouseEvent | TouchEvent) => {
      event.preventDefault();
      isDragging.value = true;

      document.addEventListener('mousemove', updateProgress);
      document.addEventListener('touchmove', updateProgress);
      document.addEventListener('mouseup', stopDrag);
      document.addEventListener('touchend', stopDrag);
    };

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
      // 參數
      minValue,
      maxValue,
      radius,
      circleFillColor,
      backgroundColor,
      progressColor,
      progress,

      // 0% 起點
      startOffsetAngleDeg,

      // 彈窗
      showRangePopup,
      openRangePopup,
      closeRangePopup,

      // 拖曳
      container,
      isDragging,
      startDrag,

      // 路徑
      backgroundPath,
      progressPath,

      // 圓點位置 / 顯示數值
      handlePosition,
      displayValue,

      // 驗證
      validateRadius,
    };
  },
});
</script>

<style scoped>
/* 可根據需求自行增減樣式 */
</style>
