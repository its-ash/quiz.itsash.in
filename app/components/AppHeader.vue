<script setup lang="ts">
const search = ref('')
const isFocused = ref(false)

const { data: pages } = await useAsyncData('header-all-pages', () =>
  queryCollection('content').all()
)

const results = computed(() => {
  const q = search.value.trim().toLowerCase()
  if (!q) return []
  return (pages.value || [])
    .filter((p: any) => p.path && p.path !== '/')
    .map((p: any) => {
      const segments = p.path.split('/').filter(Boolean)
      return {
        path: p.path,
        title: p.title || segments.at(-1) || p.path,
        category: segments[0] || '',
      }
    })
    .filter(
      (i) =>
        i.title.toLowerCase().includes(q) ||
        i.path.toLowerCase().includes(q) ||
        i.category.toLowerCase().includes(q)
    )
    .slice(0, 8)
})

function selectResult() {
  if (results.value.length > 0) {
    navigateTo(results.value[0].path)
    search.value = ''
  }
}
</script>

<template>
  <header class="sticky top-0 z-50 border-b border-c-border-light bg-c-bg/90 backdrop-blur">
    <div class="mx-auto flex max-w-[1440px] items-center justify-between gap-4 px-4 py-4 sm:px-8">
      <NuxtLink to="/" class="flex shrink-0 items-center gap-2">
        <span class="flex h-8 w-8 items-center justify-center rounded-xl bg-c-fg font-mono text-sm font-800 text-c-bg">
          &gt;_
        </span>
        <span class="font-mono text-lg font-700 tracking-tight">
          quiz<span class="text-c-muted-fg">://</span>
        </span>
      </NuxtLink>

      <div class="relative hidden max-w-md flex-1 md:flex">
        <div
          class="flex w-full items-center gap-2 rounded-xl border border-c-border-light bg-white/5 px-4 py-2 transition hover:border-white/20 hover:bg-white/10"
        >
          <svg class="h-4 w-4 shrink-0 text-c-muted-fg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="11" cy="11" r="7" />
            <line x1="21" y1="21" x2="16.65" y2="16.65" />
          </svg>
          <input
            v-model="search"
            type="text"
            placeholder="rust, closures, joins…"
            autocomplete="off"
            class="w-full flex-1 border-none bg-transparent font-mono text-sm text-c-fg placeholder:text-c-muted-fg focus:outline-none"
            @focus="isFocused = true"
            @blur="setTimeout(() => (isFocused = false), 150)"
            @keydown.enter="selectResult"
          />
        </div>

        <div
          v-if="isFocused && results.length > 0"
          class="absolute inset-x-0 top-full z-50 mt-2 overflow-hidden rounded-xl border border-c-border-light bg-c-muted shadow-[0_20px_60px_-15px_rgba(0,0,0,0.6)]"
        >
          <NuxtLink
            v-for="item in results"
            :key="item.path"
            :to="item.path"
            class="flex items-center justify-between gap-4 px-4 py-3 font-mono text-sm transition hover:bg-white/5"
          >
            <span class="truncate">{{ item.title }}</span>
            <span class="shrink-0 text-xs text-c-muted-fg">{{ item.category }}</span>
          </NuxtLink>
        </div>
      </div>

      <nav class="flex items-center gap-4 sm:gap-8">
        <NuxtLink to="/" class="hidden font-mono text-sm transition hover:opacity-70 sm:inline-block">tracks</NuxtLink>
      </nav>
    </div>
  </header>
</template>
