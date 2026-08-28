<script setup lang="ts">
const props = defineProps<{
  page: any
  slug: string
}>()

type TopicRow = {
  num: string
  title: string
  path: string
  description: string
}

type PartSection = {
  title: string
  topics: TopicRow[]
}

const categoryMeta: Record<string, { abbr: string; color: string }> = {
  rust: { abbr: 'Rs', color: '#dea584' },
  python: { abbr: 'Py', color: '#3572A5' },
  js: { abbr: 'Js', color: '#f1e05a' },
  lua: { abbr: 'Lu', color: '#000080' },
}

const meta = computed(() => categoryMeta[props.slug] || { abbr: props.slug.slice(0, 2).toUpperCase(), color: '#888888' })

function textOf(node: any): string {
  if (typeof node === 'string') return node
  if (!Array.isArray(node)) return ''
  return node
    .slice(2)
    .map((n: any) => textOf(n))
    .join('')
}

function linkOf(node: any): { text: string; href: string } | null {
  if (!Array.isArray(node)) return null
  if (node[0] === 'a') {
    return { text: textOf(node), href: node[1]?.href || '' }
  }
  for (const child of node.slice(2)) {
    const found = linkOf(child)
    if (found) return found
  }
  return null
}

const heroDescription = computed(() => {
  const value = props.page?.body?.value
  if (!Array.isArray(value)) return ''
  for (const node of value) {
    if (Array.isArray(node) && node[0] === 'p') return textOf(node)
  }
  return ''
})

const parts = computed<PartSection[]>(() => {
  const value = props.page?.body?.value
  if (!Array.isArray(value)) return []

  const result: PartSection[] = []
  let current: PartSection | null = null
  let inCurriculum = false

  for (const node of value) {
    if (!Array.isArray(node)) continue
    const [tag] = node

    if (tag === 'h2') {
      inCurriculum = textOf(node).trim().toLowerCase() === 'curriculum'
      continue
    }
    if (!inCurriculum) continue

    if (tag === 'h3') {
      current = { title: textOf(node).replace(/^Part\s*\d+\s*[—-]\s*/i, ''), topics: [] }
      result.push(current)
      continue
    }

    if (tag === 'table' && current) {
      const tbody = node.slice(2).find((n: any) => Array.isArray(n) && n[0] === 'tbody')
      const rows = tbody ? tbody.slice(2) : []
      for (const row of rows) {
        if (!Array.isArray(row)) continue
        const cells = row.slice(2)
        const num = textOf(cells[0])
        const link = linkOf(cells[1])
        const description = textOf(cells[2])
        if (link) {
          current.topics.push({ num, title: link.text, path: link.href, description })
        }
      }
    }
  }

  return result
})

const totalTopics = computed(() => parts.value.reduce((sum, p) => sum + p.topics.length, 0))
const totalQuestions = computed(() => totalTopics.value * 20)
</script>

<template>
  <div>
    <!-- ============ BREADCRUMB ============ -->
    <div class="border-b border-c-border-light">
      <div class="mx-auto max-w-[1440px] px-4 py-3 sm:px-8">
        <nav class="flex items-center gap-2 font-mono text-xs text-c-muted-fg">
          <NuxtLink to="/" class="transition hover:text-c-fg">quiz<span class="text-c-muted-fg">://</span></NuxtLink>
          <span class="text-c-muted-fg/60">/</span>
          <span class="text-c-fg">{{ slug }}</span>
        </nav>
      </div>
    </div>

    <!-- ============ PAGE HEADER ============ -->
    <section class="relative overflow-hidden border-b border-c-border-light">
      <div class="pointer-events-none absolute inset-0 grid-fade" />

      <div class="relative mx-auto max-w-[1440px] px-4 py-12 sm:px-8 md:py-16">
        <div class="flex flex-col gap-8 lg:flex-row lg:items-end lg:justify-between">
          <div class="max-w-2xl">
            <div class="mb-6 inline-flex items-center gap-2 rounded-xl border border-c-border-light bg-white/5 px-4 py-2">
              <span class="h-2 w-2 rounded-full" :style="{ backgroundColor: meta.color }" />
              <span class="font-mono text-xs">zero to hero — {{ slug }} track</span>
            </div>

            <h1 class="mb-4 flex items-center gap-4 font-mono text-3xl font-800 leading-tight tracking-tight sm:text-4xl lg:text-5xl">
              <span class="flex h-14 w-14 shrink-0 items-center justify-center rounded-xl border border-c-border-light bg-white/5 font-mono text-xl font-700">
                {{ meta.abbr }}
              </span>
              {{ slug }}
            </h1>

            <p class="mb-6 text-base text-c-fg/70 sm:text-lg">{{ heroDescription }}</p>

            <div class="flex flex-wrap items-center gap-3">
              <span class="inline-flex items-center gap-2 rounded-xl border border-c-border-light bg-c-muted px-4 py-2 font-mono text-xs text-c-fg/70">
                {{ totalTopics }} topics
              </span>
              <span class="inline-flex items-center gap-2 rounded-xl border border-c-border-light bg-c-muted px-4 py-2 font-mono text-xs text-c-fg/70">
                {{ totalQuestions }} questions
              </span>
              <span class="inline-flex items-center gap-2 rounded-xl border border-c-border-light bg-c-muted px-4 py-2 font-mono text-xs text-c-fg/70">
                {{ parts.length }} parts
              </span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ============ TOPIC LISTING ============ -->
    <main class="mx-auto max-w-[1440px] px-4 py-12 sm:px-8 md:py-16">
      <section v-for="(part, i) in parts" :key="part.title" class="mb-14" :class="{ 'mb-0': i === parts.length - 1 }">
        <div class="mb-5 flex items-baseline justify-between gap-4">
          <h2 class="font-mono text-lg font-700 sm:text-xl">
            <span class="text-c-muted-fg">Part {{ i + 1 }} —</span> {{ part.title }}
          </h2>
          <span class="font-mono text-xs text-c-muted-fg">{{ part.topics.length }} topics</span>
        </div>

        <div class="flex flex-col gap-3">
          <NuxtLink
            v-for="topic in part.topics"
            :key="topic.path"
            :to="topic.path"
            class="group flex items-center gap-4 rounded-2xl border border-c-border-light bg-c-muted px-5 py-4 transition hover:border-white/30 hover:bg-white/5 sm:gap-6 sm:px-6 sm:py-5"
          >
            <span class="flex h-10 w-10 shrink-0 items-center justify-center rounded-xl border border-c-border-light bg-white/5 font-mono text-sm font-700">{{ topic.num }}</span>
            <div class="min-w-0 flex-1">
              <h3 class="mb-1 font-mono text-base font-600">{{ topic.title }}</h3>
              <p class="truncate text-sm text-c-muted-fg">{{ topic.description }}</p>
            </div>
            <span class="hidden shrink-0 font-mono text-xs text-c-muted-fg sm:inline">20 questions</span>
            <svg class="h-5 w-5 shrink-0 text-c-muted-fg transition group-hover:-translate-y-1 group-hover:translate-x-1 group-hover:text-c-fg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="7" y1="17" x2="17" y2="7" />
              <polyline points="7 7 17 7 17 17" />
            </svg>
          </NuxtLink>
        </div>
      </section>
    </main>
  </div>
</template>
