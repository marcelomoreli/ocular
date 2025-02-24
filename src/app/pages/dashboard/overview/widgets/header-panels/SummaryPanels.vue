<template>
  <div :class="classes" @animationend="animationsDone++">
    <SummaryPanel
      :values="income"
      color="success"
      to="/income"
      :tooltip="t('page.dashboard.jumpToIncome', { year: state.activeYear })"
      :title="t('page.dashboard.income')"
      @pointerenter="emit('hoveredPanel', 'income')"
      @pointerleave="emit('hoveredPanel')"
      @pointercancel="emit('hoveredPanel')"
    />

    <SummaryPanel
      :sub-title="n(expensePercentage, 'percent')"
      to="/expenses"
      :tooltip="t('page.dashboard.jumpToExpenses', { year: state.activeYear })"
      :values="expenses"
      color="warning"
      :title="t('page.dashboard.expenses')"
      @pointerenter="emit('hoveredPanel', 'expenses')"
      @pointerleave="emit('hoveredPanel')"
      @pointercancel="emit('hoveredPanel')"
    />

    <SummaryPanel
      :sub-title="n(endingBalanceTotal ? 1 - expensePercentage : 0, 'percent')"
      :values="endingBalance"
      color="primary"
      :title="t('page.dashboard.endingBalance')"
    />

    <SummaryPanel
      :sub-title="n(remainingBalancePercentage, 'percent')"
      :alt="
        state.activeYear < time.year.value
          ? t('page.dashboard.yearInThePast')
          : state.activeYear > time.year.value
            ? t('page.dashboard.yearInTheFuture')
            : time.month.value === 11
              ? t('page.dashboard.yearEnding')
              : undefined
      "
      :values="remainingBalance"
      color="secondary"
      :title="t('page.dashboard.remainingBalance', { year: state.activeYear + 1 })"
    />
  </div>
</template>

<script lang="ts" setup>
import SummaryPanel from './SummaryPanel.vue';
import { HoveredPanel } from '@app/pages/dashboard/overview/widgets/header-panels/SummaryPanels.types.ts';
import { useTime } from '@composables';
import { useDataStore } from '@store/state';
import { totals } from '@store/state/utils/budgets';
import { aggregate, ClassNames, subtract, sum } from '@utils';
import { computed, ref, useCssModule } from 'vue';
import { useI18n } from 'vue-i18n';

const emit = defineEmits<{
  hoveredPanel: (panel?: HoveredPanel) => void;
}>();

const props = defineProps<{
  class?: ClassNames;
}>();

const { state } = useDataStore();
const { t, n } = useI18n();
const time = useTime();

const animationsDone = ref(0);
const styles = useCssModule();

const classes = computed(() => [
  props.class,
  styles.summaryPanels,
  {
    [styles.unAnimated]: animationsDone.value >= 3
  }
]);

const incomeTotals = computed(() => totals(state.income));
const expensesTotals = computed(() => totals(state.expenses));

const income = computed(() => aggregate(incomeTotals.value));
const expenses = computed(() => aggregate(expensesTotals.value));
const endingBalance = computed(() => subtract(income.value, expenses.value));
const endingBalanceTotal = computed(() => sum(endingBalance.value));

const expensePercentage = computed(() => {
  const expenses = sum(expensesTotals.value);
  const income = sum(incomeTotals.value);
  return income ? expenses / income : 0;
});

const remainingBalance = computed(() => {
  const nextMonth = time.year.value === state.activeYear ? time.month.value + 1 : 0;
  return sum(subtract(incomeTotals.value.slice(nextMonth), expensesTotals.value.slice(nextMonth)));
});

const remainingBalancePercentage = computed(() => {
  const endBalance = endingBalance.value[endingBalance.value.length - 1];
  return endBalance ? remainingBalance.value / endBalance : 0;
});
</script>

<style lang="scss" module>
@use '@styles/globals.scss';
@use 'sass:math';

.summaryPanels {
  --panels: 4;
  display: grid;
  grid-template: 1fr / repeat(var(--panels), 1fr);
  width: 100%;
  gap: 10px;

  @include globals.onMobileDevices {
    grid-template: repeat(var(--panels), 1fr) / 1fr;
    height: auto;
  }

  > * {
    opacity: 0;
    animation: var(--animation-fade-in-right) var(--transition-m) forwards;
  }

  &.unAnimated > * {
    animation: none;
    opacity: 1;
    transform: none;
  }

  @for $i from 1 through 4 {
    > *:nth-child(#{$i}) {
      animation-delay: calc(var(--transition-s) * #{math.div($i, 4)});
    }
  }
}
</style>
