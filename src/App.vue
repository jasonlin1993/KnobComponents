<template>
  <div>
    <Knob />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Knob from './components/Knob.vue';

export default defineComponent({
  name: 'App',
  components: {
    Knob, // <-- 宣告在 components
  },
  setup() {
    return {};
  },
});
</script>
