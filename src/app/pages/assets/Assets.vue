<template>
  <BudgetPane type="assets">
    <template #title>
      <YearToggle key-path="page.assets.assetsFor" />
    </template>
  </BudgetPane>
</template>

<script lang="ts" setup>
import BudgetPane from '@components/feature/BudgetPane.vue';
import YearToggle from '@components/feature/YearToggle.vue';
</script>
