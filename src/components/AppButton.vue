<script setup lang="ts">
import { Button } from "@/components/ui/button";

withDefaults(
  defineProps<{
    variant?: "default" | "outline" | "secondary" | "ghost";
    size?: "default" | "sm" | "lg" | "icon";
    disabled?: boolean;
  }>(),
  {
    variant: "outline",
    size: "default",
    disabled: false,
  },
);
</script>

<template>
  <Button
    :variant="variant"
    :size="size"
    :disabled="disabled"
    class="font-semibold transition-all duration-200"
  >
    <slot />
  </Button>
</template>

