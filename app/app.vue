<script setup lang="ts">
const route = useRoute()

useHead({
  htmlAttrs: { lang: 'en' },
  title: 'quiz — Minimalist Monochrome',
  meta: [
    { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    { name: 'theme-color', content: '#0a0a0a' },
    { name: 'charset', content: 'utf-8' },
    { name: 'robots', content: 'index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1' },
    { name: 'author', content: 'quiz' },
    { name: 'format-detection', content: 'telephone=no' },
    { 'http-equiv': 'X-UA-Compatible', content: 'IE=edge' },
  ],
  link: [
    { rel: 'canonical', href: () => 'https://quiz.itsash.in' + (route.path || '/') },
    { rel: 'icon', type: 'image/svg+xml', href: '/favicon.svg' },
    { rel: 'manifest', href: '/site.webmanifest' },
    { rel: 'apple-touch-icon', href: '/favicon.svg' },
    { rel: 'preconnect', href: 'https://fonts.googleapis.com', crossorigin: '' },
    { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: '' },
    { rel: 'dns-prefetch', href: 'https://fonts.googleapis.com' },
  ],
  script: [
    {
      async: true,
      src: 'https://www.googletagmanager.com/gtag/js?id=G-QHKG9XZ6RC',
    },
    {
      innerHTML: `window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
gtag('js', new Date());
gtag('config', 'G-QHKG9XZ6RC');`,
    },
    {
      type: 'application/ld+json',
      innerHTML: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'WebSite',
        name: 'quiz',
        url: 'https://quiz.itsash.in',
        description: 'A personal reading space built with Nuxt Content — Rust curriculum, programming guides, and technical documentation.',
        potentialAction: {
          '@type': 'SearchAction',
          target: 'https://quiz.itsash.in/?q={search_term_string}',
          'query-input': 'required name=search_term_string',
        },
      }),
    },
  ],
})

useSeoMeta({
  ogSiteName: 'quiz',
  ogType: 'website',
  twitterCard: 'summary_large_image',
  twitterSite: '@quiz',
})
</script>

<template>
  <div class="min-h-screen bg-c-bg font-sans text-c-fg antialiased">
    <AppHeader />

    <main class="relative z-10">
      <NuxtPage />
    </main>

    <footer class="relative z-10 border-t border-c-border-light">
      <div class="mx-auto flex max-w-[1440px] flex-col items-center justify-between gap-8 px-4 py-12 sm:flex-row sm:px-8">
        <div class="flex items-center gap-2">
          <span class="flex h-8 w-8 items-center justify-center rounded-xl bg-c-fg font-mono text-sm font-800 text-c-bg">
            &gt;_
          </span>
          <span class="font-mono text-sm font-700">
            quiz<span class="text-c-muted-fg">://</span>
          </span>
          <span class="ml-4 font-mono text-xs text-c-muted-fg">&copy; {{ new Date().getFullYear() }} — all rights reserved</span>
        </div>
        <div class="flex items-center gap-8">
          <a href="https://its-ash.github.io/" target="_blank" rel="noopener noreferrer" class="font-mono text-xs text-c-muted-fg transition hover:text-c-fg">
            its-ash.github.io
          </a>
        </div>
      </div>
    </footer>
  </div>
</template>