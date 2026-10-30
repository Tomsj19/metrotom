<script setup lang="ts">
defineProps<{
  bpm: number
}>()
</script>

<template>
  <div class="flex items-baseline gap-2">
    <span
      class="text-7xl tracking-tight tabular-nums sm:text-8xl"
    >
      {{ bpm }}
    </span>

    <span
      class="mt-1 text-sm font-medium uppercase tracking-[0.2em] text-muted-foreground"
    >
      BPM
    </span>
  </div>
</template>