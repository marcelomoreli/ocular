<template>
  <Transition
    :enter-active-class="$style.transitionTarget"
    :enter-from-class="$style.transitionOrigin"
    :leave-active-class="$style.transitionTarget"
    :leave-to-class="$style.transitionOrigin"
    mode="out-in"
  >
    <component :is="is" />
  </Transition>
</template>

<script setup lang="ts">
defineProps<{
  is: object;
}>();
</script>

<style lang="scss" module>
.transitionTarget {
  transition: all var(--transition-s);
}

.transitionOrigin {
  opacity: 0;
}
</style>
