<script setup lang="ts">
const props = defineProps<{
  error: {
    statusCode: number
    statusMessage?: string
    message?: string
  }
}>()

const handleError = () => clearError({ redirect: '/' })
</script>

<template>
  <div class="flex min-h-screen flex-col items-center justify-center bg-c-bg px-6 font-sans text-c-fg antialiased">
    <div class="flex flex-col items-center gap-8 text-center">
      <div class="flex items-center gap-2 rounded-xl border border-c-border-light bg-white/5 px-4 py-2">
        <span class="h-2 w-2 rounded-full bg-[#f87171]" />
        <span class="font-mono text-xs">error</span>
      </div>

      <h1 class="font-mono text-[6rem] font-800 tracking-tight text-c-fg sm:text-[8rem]" style="line-height: 0.9">
        {{ error?.statusCode || 500 }}
      </h1>

      <p class="max-w-md font-mono text-xl font-700 tracking-tight text-c-fg sm:text-2xl">
        <template v-if="error?.statusCode === 404">
          Page not found
        </template>
        <template v-else>
          {{ error?.statusMessage || 'Something went wrong' }}
        </template>
      </p>

      <p class="max-w-md text-base leading-relaxed text-c-muted-fg">
        <template v-if="error?.statusCode === 404">
          The page you're looking for doesn't exist or has been moved.
        </template>
        <template v-else>
          An unexpected error occurred while rendering this page.
        </template>
      </p>

      <div class="flex flex-wrap items-center justify-center gap-4">
        <button
          type="button"
          class="inline-flex items-center gap-2 rounded-xl bg-c-fg px-6 py-3 font-mono text-sm font-600 text-c-bg transition hover:opacity-90"
          @click="handleError"
        >
          <span>&larr;</span>
          <span>Back home</span>
        </button>
        <NuxtLink
          to="/"
          class="inline-flex items-center gap-2 rounded-xl border border-c-border-light px-6 py-3 font-mono text-sm font-600 text-c-fg transition hover:border-white/30 hover:bg-white/5"
        >
          <span>Browse tracks</span>
        </NuxtLink>
      </div>
    </div>
  </div>
</template>