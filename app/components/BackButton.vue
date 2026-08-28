<script setup lang="ts">
withDefaults(defineProps<{
  to?: string
  label?: string
  fallback?: boolean
}>(), {
  label: 'Back',
  fallback: true,
})

const router = useRouter()

function goBack() {
  if (window.history.length > 1) {
    router.back()
  } else {
    router.push('/')
  }
}
</script>

<template>
  <component
    :is="to ? NuxtLink : 'button'"
    v-bind="to ? { to } : { type: 'button', onClick: goBack }"
    class="group inline-flex items-center gap-2 rounded-xl border border-c-border-light px-5 py-2.5 font-mono text-xs font-medium text-c-fg transition-colors duration-100 hover:border-white/30 hover:bg-white/5 focus-visible:outline-2 focus-visible:outline-offset-2"
  >
    <span>&larr;</span>
    <span>{{ label }}</span>
  </component>
</template>