<template>
  <BudgetPane type="shares">
    <template #title>
      <YearToggle key-path="page.shares.sharesFor" />
    </template>
  </BudgetPane>
</template>

<script lang="ts" setup>
import BudgetPane from '@components/feature/BudgetPane.vue';
import YearToggle from '@components/feature/YearToggle.vue';
</script>
