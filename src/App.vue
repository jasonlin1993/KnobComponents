<template>
  <div class="flex h-screen w-full items-center justify-center">
    <div
      class="relative h-64 w-64 select-none"
      @mousedown="startDrag"
      @touchstart="startDrag"
      ref="container"
    >
      <svg class="h-full w-full" viewBox="0 0 100 100">
        <circle cx="50" cy="50" r="45" fill="none" stroke="#e2e8f0" stroke-width="8" />
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
        <!-- Draggable handle -->
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
      <div
        class="absolute inset-0 flex items-center justify-center text-4xl font-bold text-gray-800"
        aria-live="polite"
      >
        {{ Math.round(progress) }}
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted } from 'vue';

export default defineComponent({
  name: 'EnhancedRotatingProgress',
  setup() {
    const progress = ref(0);
    const container = ref<HTMLElement | null>(null);
    const isDragging = ref(false);
    const startAngle = ref(0);

    const radius = 45;
    const circumference = computed(() => 2 * Math.PI * radius);
    const dashOffset = computed(
      () => circumference.value - (progress.value / 100) * circumference.value,
    );

    const handlePosition = computed(() => {
      const angle = (progress.value / 100) * 2 * Math.PI - Math.PI / 2;
      return {
        x: 50 + 45 * Math.cos(angle),
        y: 50 + 45 * Math.sin(angle),
      };
    });

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

    const updateProgress = (event: MouseEvent | TouchEvent) => {
      if (!isDragging.value) return;

      const currentAngle = calculateAngle(event);
      let angleDiff = currentAngle - startAngle.value;

      if (angleDiff > 180) angleDiff -= 360;
      if (angleDiff < -180) angleDiff += 360;

      let newProgress = progress.value + (angleDiff / 360) * 100;

      if (newProgress < 0) newProgress += 100;
      if (newProgress > 100) newProgress -= 100;

      progress.value = newProgress;
      startAngle.value = currentAngle;
    };

    const startDrag = (event: MouseEvent | TouchEvent) => {
      event.preventDefault();
      isDragging.value = true;
      startAngle.value = calculateAngle(event);
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
      progress,
      container,
      circumference,
      dashOffset,
      handlePosition,
      startDrag,
    };
  },
});
</script>
