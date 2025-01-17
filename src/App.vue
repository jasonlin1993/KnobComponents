<template>
  <div class="flex h-screen items-center justify-center">
    <div
      class="relative flex h-40 w-40 items-center justify-center"
      @mousemove="handleMouseMove"
      @mouseleave="resetProgress"
    >
      <!-- Circle Progress -->
      <svg class="absolute h-full w-full -rotate-90 transform">
        <circle cx="50%" cy="50%" r="45%" fill="none" stroke="#e5e7eb" stroke-width="10"></circle>
        <circle
          cx="50%"
          cy="50%"
          r="45%"
          fill="none"
          :stroke="progressColor"
          stroke-width="10"
          stroke-dasharray="282.6"
          :stroke-dashoffset="dashOffset"
          stroke-linecap="round"
        ></circle>
      </svg>

      <!-- Display Progress -->
      <div class="text-center text-gray-700">
        <span class="text-2xl font-bold">{{ progress }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

export default defineComponent({
  name: 'CircularProgress',
  setup() {
    const progress = ref(0);

    // 計算進度顏色
    const progressColor = computed(() => {
      if (progress.value < 50) return '#10b981'; // 綠色
      if (progress.value < 75) return '#fbbf24'; // 黃色
      return '#ef4444'; // 紅色
    });

    // 圓周的長度與進度偏移量
    const circumference = 2 * Math.PI * 45; // 半徑 45%
    const dashOffset = computed(() => {
      return circumference - (progress.value / 100) * circumference;
    });

    const handleMouseMove = (event: MouseEvent) => {
      const target = event.currentTarget as HTMLElement;
      const rect = target.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;
      const centerX = rect.width / 2;
      const centerY = rect.height / 2;

      // 計算滑鼠相對於圓心的角度，轉為進度值
      const angle = (Math.atan2(y - centerY, x - centerX) * 180) / Math.PI + 180;
      progress.value = Math.round((angle / 360) * 100);
    };

    const resetProgress = () => {
      progress.value = 0;
    };

    return {
      progress,
      progressColor,
      dashOffset,
      handleMouseMove,
      resetProgress,
    };
  },
});
</script>
