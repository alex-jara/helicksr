<script setup lang="ts">
import { PropType } from 'vue';

defineProps({
  type: {
    type: String as PropType<'button' | 'submit' | 'reset'>,
    default: 'button',
  },
});
</script>

<template>
  <button
    :type="type"
    class="inline-flex items-center px-4 py-2 bg-white dark:bg-gray-300 border border-gray-300 rounded-md font-semibold text-xs text-gray-700 uppercase tracking-widest shadow-sm hover:text-gray-500 focus:outline-none active:text-gray-800 active:bg-gray-50 disabled:opacity-25 transition"
  >
    <slot />
  </button>
</template>
