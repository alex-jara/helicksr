<script setup lang="ts">
import { ApiTokenManager } from './Partials';
import { AppLayout } from '~~/Layouts';
import { PropType } from 'vue';
import { ApiToken, CRUDPermissions } from '~~/types';

defineProps({
  tokens: {
    type: Array as PropType<ApiToken[]>,
    default: () => [],
  },
  availablePermissions: {
    type: Array as PropType<CRUDPermissions[]>,
    default: () => [],
  },
  defaultPermissions: {
    type: Array as PropType<CRUDPermissions[]>,
    default: () => ['read'],
  },
});
</script>

<template>
  <AppLayout title="API Tokens">
    <template #header>
      <h2 class="font-semibold text-xl text-gray-800 leading-tight">
        API Tokens
      </h2>
    </template>

    <div>
      <div class="max-w-7xl mx-auto py-10 sm:px-6 lg:px-8">
        <ApiTokenManager
          :tokens="tokens"
          :available-permissions="availablePermissions"
          :default-permissions="defaultPermissions"
        />
      </div>
    </div>
  </AppLayout>
</template>
