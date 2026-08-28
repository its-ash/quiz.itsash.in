<script setup lang="ts">
const { data: pages } = await useAsyncData('all-pages', () =>
  queryCollection('content').all()
)

useSeoMeta({
  title: 'quiz:// — pick a track',
  description: 'Zero-to-hero quiz tracks for the languages you actually ship — Rust, Python, JavaScript, Lua, and more. Production-grade gotchas, not toy snippets.',
  ogTitle: 'quiz:// — pick a track',
  ogDescription: 'Zero-to-hero quiz tracks for the languages you actually ship.',
  ogType: 'website',
  robots: 'index, follow',
})

type CategoryMeta = {
  abbr: string
  color: string
  level: string
}

const categoryMeta: Record<string, CategoryMeta> = {
  rust: { abbr: 'Rs', color: '#dea584', level: 'advanced' },
  python: { abbr: 'Py', color: '#3572A5', level: 'all levels' },
  js: { abbr: 'Js', color: '#f1e05a', level: 'all levels' },
  lua: { abbr: 'Lu', color: '#000080', level: 'new' },
}

function firstParagraphText(body: any): string {
  const value = body?.value
  if (!Array.isArray(value)) return ''
  for (const node of value) {
    if (Array.isArray(node) && node[0] === 'p') {
      return node
        .slice(2)
        .map((n: any) => (typeof n === 'string' ? n : Array.isArray(n) ? n.at(-1) : ''))
        .join('')
    }
  }
  return ''
}

const categories = computed(() => {
  const indexPages = (pages.value || []).filter((p: any) => {
    const segments = p.path.split('/').filter(Boolean)
    return segments.length === 1 && p.path !== '/'
  })

  return indexPages
    .map((p: any) => {
      const slug = p.path.replace(/^\//, '')
      const topicCount = (pages.value || []).filter((t: any) => {
        const segments = t.path.split('/').filter(Boolean)
        return segments.length === 2 && segments[0] === slug
      }).length
      const meta = categoryMeta[slug] || { abbr: slug.slice(0, 2).toUpperCase(), color: '#888888', level: 'mixed' }
      return {
        slug,
        path: p.path,
        title: p.title?.replace(/:\s*Zero to Hero$/i, '') || slug,
        description: firstParagraphText(p.body),
        topicCount,
        questionCount: topicCount * 20,
        ...meta,
      }
    })
    .sort((a, b) => b.topicCount - a.topicCount)
})

const totalTopics = computed(() => categories.value.reduce((sum, c) => sum + c.topicCount, 0))
const totalQuestions = computed(() => categories.value.reduce((sum, c) => sum + c.questionCount, 0))
</script>

<template>
  <div>
    <!-- ============ HERO ============ -->
    <section class="relative overflow-hidden border-b border-c-border-light">
      <div class="pointer-events-none absolute inset-0 grid-fade" />

      <div class="relative mx-auto max-w-[1440px] px-4 py-16 sm:px-8 md:py-24">
        <div class="grid grid-cols-1 items-center gap-12 lg:grid-cols-2">
          <div>
            <div class="mb-8 inline-flex items-center gap-2 rounded-xl border border-c-border-light bg-white/5 px-4 py-2">
              <span class="pulse-dot h-2 w-2 rounded-full bg-[#00ff9d]" />
              <span class="font-mono text-xs">{{ categories.length }} tracks live · {{ totalQuestions }}+ questions · updated weekly</span>
            </div>

            <h1 class="mb-8 font-mono text-4xl font-800 leading-tight tracking-tight sm:text-5xl lg:text-6xl">
              Master any language,<br />
              one
              <span class="relative inline-block">
                <span class="relative z-10">gotcha</span>
                <span class="absolute inset-x-0 bottom-2 -z-0 h-4 bg-white/10" />
              </span>
              at a time.
            </h1>

            <p class="mb-8 max-w-lg text-lg text-c-muted-fg">
              Every track runs foundations &rarr; intermediate &rarr; advanced &rarr; expert.
              No toy snippets — ownership traps, type coercion surprises, off-by-ones,
              and the production bugs that actually bite.
            </p>

            <div class="flex flex-wrap items-center gap-4">
              <a href="#tracks" class="inline-flex items-center gap-2 rounded-xl bg-c-fg px-6 py-4 font-mono text-sm font-600 text-c-bg transition hover:opacity-90">
                Browse tracks
                <svg class="h-4 w-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="5" y1="12" x2="19" y2="12" />
                  <polyline points="12 5 19 12 12 19" />
                </svg>
              </a>
            </div>

            <div class="mt-12 grid max-w-md grid-cols-3 gap-6">
              <div>
                <p class="font-mono text-2xl font-800">{{ categories.length }}</p>
                <p class="mt-1 font-mono text-xs text-c-muted-fg">tracks</p>
              </div>
              <div>
                <p class="font-mono text-2xl font-800">{{ totalTopics }}</p>
                <p class="mt-1 font-mono text-xs text-c-muted-fg">topics</p>
              </div>
              <div>
                <p class="font-mono text-2xl font-800">{{ totalQuestions }}</p>
                <p class="mt-1 font-mono text-xs text-c-muted-fg">questions</p>
              </div>
            </div>
          </div>

          <div class="relative">
            <div class="overflow-hidden rounded-2xl border border-c-border-light bg-c-muted shadow-[0_0_60px_-15px_rgba(255,255,255,0.08)]">
              <div class="flex items-center gap-2 border-b border-c-border-light bg-white/5 px-4 py-4">
                <span class="h-3 w-3 rounded-full bg-[#ff5f56]" />
                <span class="h-3 w-3 rounded-full bg-[#ffbd2e]" />
                <span class="h-3 w-3 rounded-full bg-[#27c93f]" />
                <span class="ml-2 font-mono text-xs text-c-muted-fg">curriculum.map</span>
              </div>
              <div class="space-y-5 p-6 sm:p-8">
                <div class="flex items-center gap-4">
                  <span class="flex h-9 w-9 shrink-0 items-center justify-center rounded-xl bg-c-fg font-mono text-xs font-700 text-c-bg">01</span>
                  <div class="flex-1">
                    <p class="font-mono text-sm font-600">Foundations</p>
                    <p class="font-mono text-xs text-c-muted-fg">syntax, types, control flow</p>
                  </div>
                  <svg class="h-4 w-4 text-c-fg/70" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12" /></svg>
                </div>
                <div class="flex items-center gap-4">
                  <span class="flex h-9 w-9 shrink-0 items-center justify-center rounded-xl bg-c-fg font-mono text-xs font-700 text-c-bg">02</span>
                  <div class="flex-1">
                    <p class="font-mono text-sm font-600">Intermediate</p>
                    <p class="font-mono text-xs text-c-muted-fg">collections, functions, errors</p>
                  </div>
                  <svg class="h-4 w-4 text-c-fg/70" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12" /></svg>
                </div>
                <div class="flex items-center gap-4">
                  <span class="flex h-9 w-9 shrink-0 items-center justify-center rounded-xl border border-c-border-light bg-white/5 font-mono text-xs font-700">03</span>
                  <div class="flex-1">
                    <p class="font-mono text-sm font-600">Advanced</p>
                    <p class="font-mono text-xs text-c-muted-fg">concurrency, generics, memory</p>
                  </div>
                  <span class="pulse-dot h-2 w-2 rounded-full bg-[#00ff9d]" />
                </div>
                <div class="flex items-center gap-4 opacity-40">
                  <span class="flex h-9 w-9 shrink-0 items-center justify-center rounded-xl border border-c-border-light bg-white/5 font-mono text-xs font-700">04</span>
                  <div class="flex-1">
                    <p class="font-mono text-sm font-600">Expert</p>
                    <p class="font-mono text-xs text-c-muted-fg">unsafe, macros, perf tuning</p>
                  </div>
                  <svg class="h-4 w-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="5" y="11" width="14" height="9" rx="2" /><path d="M8 11V7a4 4 0 0 1 8 0v4" /></svg>
                </div>
                <div class="flex items-center gap-1 border-t border-c-border-light pt-4 font-mono text-xs text-c-muted-fg">
                  <span>&gt;</span>
                  <span>pick a track to begin</span>
                </div>
              </div>
            </div>

            <div class="absolute -bottom-6 -left-6 hidden items-center gap-4 rounded-2xl border border-c-border-light bg-c-muted px-6 py-4 shadow-[0_0_40px_-10px_rgba(255,255,255,0.15)] sm:flex">
              <div>
                <p class="font-mono text-2xl font-800">4</p>
                <p class="font-mono text-xs text-c-muted-fg">phases per track</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ============ CATEGORY GRID ============ -->
    <main id="tracks" class="mx-auto max-w-[1440px] px-4 py-16 sm:px-8 md:py-24">
      <div class="mb-8 flex flex-col gap-4 sm:flex-row sm:items-end sm:justify-between">
        <div>
          <p class="mb-2 font-mono text-sm text-c-muted-fg">## tracks</p>
          <h2 class="font-mono text-2xl font-700 sm:text-3xl">Pick a language</h2>
        </div>
        <p class="font-mono text-sm text-c-muted-fg">{{ categories.length }} tracks · sorted by topic count</p>
      </div>

      <div class="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
        <NuxtLink
          v-for="cat in categories"
          :key="cat.slug"
          :to="cat.path"
          class="group relative flex flex-col gap-6 rounded-2xl border border-c-border-light bg-c-muted p-6 transition hover:border-white/30 hover:bg-white/5"
        >
          <div class="flex items-start justify-between">
            <span class="flex h-12 w-12 items-center justify-center rounded-xl border border-c-border-light bg-white/5 font-mono text-lg font-700">{{ cat.abbr }}</span>
            <div class="flex items-center gap-2">
              <span class="rounded-lg border border-c-border-light px-2 py-1 font-mono text-[10px] text-c-muted-fg">{{ cat.level }}</span>
              <svg class="h-5 w-5 text-c-muted-fg transition group-hover:-translate-y-1 group-hover:translate-x-1 group-hover:text-c-fg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <line x1="7" y1="17" x2="17" y2="7" />
                <polyline points="7 7 17 7 17 17" />
              </svg>
            </div>
          </div>
          <div>
            <h3 class="mb-2 font-mono text-xl font-700">{{ cat.slug }}</h3>
            <p class="text-sm text-c-fg/70">{{ cat.description }}</p>
          </div>
          <div class="flex items-center justify-between border-t border-c-border-light pt-6">
            <div class="flex items-center gap-2">
              <span class="h-3 w-3 rounded-full" :style="{ backgroundColor: cat.color }" />
              <span class="font-mono text-xs text-c-muted-fg">{{ cat.topicCount }} {{ cat.topicCount === 1 ? 'topic' : 'topics' }}</span>
            </div>
            <span class="font-mono text-xs text-c-muted-fg">{{ cat.questionCount }} questions</span>
          </div>
        </NuxtLink>
      </div>
    </main>

    <!-- ============ WHY THIS FORMAT ============ -->
    <section class="relative overflow-hidden border-t border-c-border-light">
      <div class="relative mx-auto max-w-[1440px] px-4 py-16 sm:px-8 md:py-20">
        <p class="mb-2 font-mono text-sm text-c-muted-fg">## why quiz://</p>
        <h2 class="mb-10 font-mono text-2xl font-700 sm:text-3xl">Built for gotchas, not trivia</h2>

        <div class="grid grid-cols-1 gap-6 sm:grid-cols-3">
          <div class="rounded-2xl border border-c-border-light bg-c-muted p-6">
            <span class="mb-4 flex h-10 w-10 items-center justify-center rounded-xl border border-c-border-light bg-white/5">
              <svg class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2l3 7h7l-5.5 4.5L18 21l-6-4-6 4 1.5-7.5L2 9h7z" /></svg>
            </span>
            <h3 class="mb-2 font-mono text-sm font-600">Production-grade scenarios</h3>
            <p class="text-sm text-c-muted-fg">Every question is drawn from real bugs, not toy snippets — the kind that survive code review and bite in prod.</p>
          </div>
          <div class="rounded-2xl border border-c-border-light bg-c-muted p-6">
            <span class="mb-4 flex h-10 w-10 items-center justify-center rounded-xl border border-c-border-light bg-white/5">
              <svg class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 18l6-6-6-6" /></svg>
            </span>
            <h3 class="mb-2 font-mono text-sm font-600">Progressive difficulty</h3>
            <p class="text-sm text-c-muted-fg">Basic recall &rarr; edge cases &rarr; best-practice judgment calls, ordered so every quiz builds on the last.</p>
          </div>
          <div class="rounded-2xl border border-c-border-light bg-c-muted p-6">
            <span class="mb-4 flex h-10 w-10 items-center justify-center rounded-xl border border-c-border-light bg-white/5">
              <svg class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" /><path d="M8 12l3 3 5-6" /></svg>
            </span>
            <h3 class="mb-2 font-mono text-sm font-600">Explanations that teach</h3>
            <p class="text-sm text-c-muted-fg">Collapsed answers explain the mechanism — evaluation order, coercion rules, borrow rules — not just the label.</p>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
