<!-- eslint-disable vue/no-v-html -->
<template>
  <div>
    <NuxtLoadingIndicator />

    <Header v-if="!$route.path.startsWith('/examples')" :links="links" />

    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>

    <Footer v-if="!$route.path.startsWith('/examples')" />

    <ClientOnly>
      <LazyUDocsSearch ref="searchRef" :files="files" :navigation="navigation" :links="links" :fuse="{ resultLimit: 1000 }" />
    </ClientOnly>

    <UNotifications>
      <template #title="{ title }">
        <span v-html="title" />
      </template>
    </UNotifications>
    <UModals />
  </div>
</template>

<script setup lang="ts">
import { withoutTrailingSlash } from 'ufo'
import { debounce } from 'perfect-debounce'
import type { ParsedContent } from '@nuxt/content/dist/runtime/types'

const searchRef = ref()

const route = useRoute()
const colorMode = useColorMode()
const { branch } = useContentSource()

const { data: nav } = await useAsyncData('navigation', () => fetchContentNavigation())
const { data: files } = useLazyFetch<ParsedContent[]>('/api/search.json', { default: () => [], server: false })

// Computed

const navigation = computed(() => {
  if (branch.value?.name === 'dev') {
    const dev = nav.value.find(item => item._path === '/dev')?.children
    const pro = nav.value.find(item => item._path === '/pro')

    return [
      ...dev,
      ...(pro ? [pro] : [])
    ]
  }

  return nav.value.filter(item => item._path !== '/dev')
})

const color = computed(() => colorMode.value === 'dark' ? '#18181b' : 'white')

const links = computed(() => {
  return [{
    label: 'Docs',
    icon: 'i-heroicons-book-open',
    to: branch.value?.name === 'dev' ? '/dev/getting-started' : '/getting-started',
    active: branch.value?.name === 'dev' ? (route.path.startsWith('/dev/getting-started') || route.path.startsWith('/dev/components')) : (route.path.startsWith('/getting-started') || route.path.startsWith('/components'))
  }, ...(navigation.value.find(item => item._path === '/pro') ? [{
    label: 'Pro',
    icon: 'i-heroicons-square-3-stack-3d',
    to: '/pro',
    active: route.path.startsWith('/pro/getting-started') || route.path.startsWith('/pro/components') || route.path.startsWith('/pro/prose')
  }, {
    label: 'Pricing',
    icon: 'i-heroicons-credit-card',
    to: '/pro/pricing'
  }, {
    label: 'Templates',
    icon: 'i-heroicons-computer-desktop',
    to: '/pro/templates'
  }] : []), {
    label: 'Releases',
    icon: 'i-heroicons-rocket-launch',
    to: '/releases'
  }].filter(Boolean)
})

// Watch

watch(() => searchRef.value?.commandPaletteRef?.query, debounce((query: string) => {
  if (!query) {
    return
  }

  useTrackEvent('Search', { props: { query: `${query} - ${searchRef.value?.commandPaletteRef.results.length} results` } })
}, 500))

// Head

useHead({
  meta: [
    { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    { key: 'theme-color', name: 'theme-color', content: color }
  ],
  link: [
    { rel: 'icon', type: 'image/svg+xml', href: '/icon.svg' },
    { rel: 'canonical', href: `https://ui.nuxt.com${withoutTrailingSlash(route.path)}` }
  ],
  htmlAttrs: {
    lang: 'en'
  }
})

useServerSeoMeta({
  ogSiteName: 'Nuxt UI',
  twitterCard: 'summary_large_image'
})

// Provide

provide('navigation', navigation)
provide('files', files)
</script>
