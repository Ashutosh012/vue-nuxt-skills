# Nuxt 3 Skill

## Description
Use this skill when the task involves **building or modifying a Nuxt 3 application** — including pages, layouts, server API routes, middleware, plugins, SEO, data fetching, SSR/SSG configuration, or Nuxt modules.

Trigger phrases: "create a page", "add an API route", "set up server route", "configure Nuxt", "add SEO meta", "create middleware", "set up auth", "add a layout", "configure rendering", "write a Nuxt plugin", "fetch data in Nuxt", "create a Nitro route", "add a Nuxt module".

---

## How to Use This Skill

When working on a Nuxt 3 task, follow this workflow:

1. **Identify the rendering requirement first** — SSR, SSG, ISR, or CSR?
2. **Determine where data lives** — server route, external API, or static?
3. **Choose the right data fetching primitive** — `useFetch`, `useAsyncData`, or `$fetch`
4. **Implement server route if needed** — validate input, handle errors, return typed data
5. **Build the page/component** — use auto-imports, no manual imports for Nuxt utils
6. **Add SEO meta** — always use `useSeoMeta()`
7. **Apply middleware** — auth, redirects, analytics

---

## Project Structure Reference

```
.
├── app.vue                        # Global app wrapper (optional)
├── nuxt.config.ts                 # Central config
├── error.vue                      # Custom error page
│
├── pages/
│   ├── index.vue                  # → /
│   ├── about.vue                  # → /about
│   ├── blog/
│   │   ├── index.vue              # → /blog
│   │   └── [slug].vue             # → /blog/:slug
│   └── users/
│       ├── index.vue              # → /users
│       ├── [id]/
│       │   ├── index.vue          # → /users/:id
│       │   └── settings.vue      # → /users/:id/settings
│       └── [...slug].vue          # → /users/* (catch-all)
│
├── components/
│   ├── ui/                        # Base: Button, Input, Modal
│   ├── App/                       # App-level: AppHeader, AppFooter
│   └── [Feature]/                 # Feature-specific components
│
├── composables/                   # Auto-imported useXxx.ts files
├── utils/                         # Auto-imported utility functions
│
├── layouts/
│   ├── default.vue                # Default layout
│   └── dashboard.vue              # Dashboard layout
│
├── middleware/
│   ├── auth.ts                    # Named: requiresAuth pages
│   └── redirect.global.ts        # Global: runs on every navigation
│
├── plugins/
│   ├── analytics.client.ts        # Client-only plugin
│   └── error-handler.ts          # Universal plugin
│
├── server/
│   ├── api/                       # → /api/* routes
│   │   ├── users/
│   │   │   ├── index.get.ts      # GET  /api/users
│   │   │   ├── index.post.ts     # POST /api/users
│   │   │   └── [id].get.ts      # GET  /api/users/:id
│   │   └── health.get.ts        # GET  /api/health
│   ├── routes/                    # Custom non-/api/* routes
│   ├── middleware/                # Server middleware (runs every request)
│   └── utils/                    # Server-only shared utilities (auto-imported)
│
├── stores/                        # Pinia stores
└── types/                         # Shared TypeScript types
```

---

## `nuxt.config.ts` Skill

### Full production-ready config template

```ts
export default defineNuxtConfig({
  devtools: { enabled: true },

  // --- Modules ---
  modules: [
    '@nuxtjs/tailwindcss',
    '@pinia/nuxt',
    '@nuxt/image',
    '@vueuse/nuxt',
    '@nuxtjs/i18n',
    'nuxt-security',
    '@nuxt/content',       // If using CMS/markdown
  ],

  // --- Runtime Config ---
  runtimeConfig: {
    // 🔒 Server-only (never sent to browser)
    databaseUrl: process.env.DATABASE_URL,
    jwtSecret: process.env.JWT_SECRET,
    stripeSecretKey: process.env.STRIPE_SECRET_KEY,
    // 🌐 Public (exposed to browser — safe values only)
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE ?? '/api',
      appName: process.env.NUXT_PUBLIC_APP_NAME ?? 'My App',
      sentryDsn: process.env.NUXT_PUBLIC_SENTRY_DSN,
    },
  },

  // --- Rendering Strategy Per Route ---
  routeRules: {
    '/':                  { prerender: true },          // Static home
    '/about':             { prerender: true },          // Static page
    '/blog/**':           { isr: 3600 },               // ISR — 1hr revalidation
    '/docs/**':           { prerender: true },          // Full static
    '/dashboard/**':      { ssr: false },               // SPA — client-only
    '/admin/**':          { ssr: false },               // SPA — client-only
    '/api/**':            { cors: true, cache: false }, // API routes
  },

  // --- TypeScript ---
  typescript: {
    strict: true,
    typeCheck: true,
  },

  // --- App Head Defaults ---
  app: {
    head: {
      charset: 'utf-8',
      viewport: 'width=device-width, initial-scale=1',
    },
  },
})
```

---

## Data Fetching Skill

### Decision tree — which primitive to use?

```
Is this inside a Vue component or page?
  ├─ YES → Is it needed for SSR (visible on first load)?
  │         ├─ YES → useFetch() or useAsyncData()
  │         └─ NO  → $fetch() inside onMounted() or with lazy: true
  └─ NO (inside event handler / store action / server route)
           └─ $fetch()
```

### `useFetch` — standard SSR data fetching

```ts
// Basic
const { data, status, error, refresh } = await useFetch<User[]>('/api/users')

// With options
const { data: user } = await useFetch<User>(`/api/users/${route.params.id}`, {
  // Cache key — must be unique per fetch
  key: `user-${route.params.id}`,
  // Re-fetch when this reactive value changes
  watch: [() => route.params.id],
  // Transform response
  transform: (res) => res.data,
  // Don't block navigation — load in background
  lazy: true,
  // Pass auth header
  headers: { Authorization: `Bearer ${token.value}` },
})

// Always handle status
if (status.value === 'error') {
  // handle error.value
}
```

### `useAsyncData` — multi-source or custom logic

```ts
const { data } = await useAsyncData('dashboard', async () => {
  const [stats, recentOrders, topProducts] = await Promise.all([
    $fetch<Stats>('/api/dashboard/stats'),
    $fetch<Order[]>('/api/orders/recent'),
    $fetch<Product[]>('/api/products/top'),
  ])
  return { stats, recentOrders, topProducts }
}, {
  watch: [selectedDateRange],
})
```

### `$fetch` — mutations and event handlers

```ts
// In component
async function createPost() {
  const post = await $fetch<Post>('/api/posts', {
    method: 'POST',
    body: {
      title: form.title,
      content: form.content,
    },
  })
  await navigateTo({ name: 'PostDetail', params: { id: post.id } })
}
```

---

## Server Routes Skill

### File naming convention

| Filename | HTTP Method | URL |
|---|---|---|
| `users/index.get.ts` | GET | `/api/users` |
| `users/index.post.ts` | POST | `/api/users` |
| `users/[id].get.ts` | GET | `/api/users/:id` |
| `users/[id].patch.ts` | PATCH | `/api/users/:id` |
| `users/[id].delete.ts` | DELETE | `/api/users/:id` |

### Full server route template

```ts
// server/api/posts/index.post.ts
import { z } from 'zod'

const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(10),
  tags: z.array(z.string()).optional().default([]),
  published: z.boolean().default(false),
})

export default defineEventHandler(async (event) => {
  // 1. Authenticate
  const user = await requireAuth(event) // from server/utils/auth.ts

  // 2. Read and validate body
  const rawBody = await readBody(event)
  const result = CreatePostSchema.safeParse(rawBody)

  if (!result.success) {
    throw createError({
      statusCode: 422,
      message: 'Validation failed',
      data: result.error.flatten(),
    })
  }

  const body = result.data

  // 3. Business logic
  const post = await db.post.create({
    data: {
      ...body,
      authorId: user.id,
    },
  })

  // 4. Return — Nitro auto-serializes
  return post
})
```

### Reading route params, query, and body

```ts
export default defineEventHandler(async (event) => {
  // Route param from [id].get.ts
  const id = getRouterParam(event, 'id')

  // Query string ?page=1&limit=20
  const query = getQuery(event)
  const page = Number(query.page ?? 1)
  const limit = Number(query.limit ?? 20)

  // Request body (POST/PATCH)
  const body = await readBody(event)

  // Headers
  const authHeader = getHeader(event, 'authorization')

  // Cookies
  const sessionToken = getCookie(event, 'session')
})
```

### Server utility (shared across routes)

```ts
// server/utils/auth.ts — auto-imported in server routes
import type { H3Event } from 'h3'

export async function requireAuth(event: H3Event) {
  const token = getCookie(event, 'auth-token')
    ?? getHeader(event, 'authorization')?.replace('Bearer ', '')

  if (!token) {
    throw createError({ statusCode: 401, message: 'Authentication required' })
  }

  try {
    const config = useRuntimeConfig()
    const payload = verifyJwt(token, config.jwtSecret)
    return payload
  } catch {
    throw createError({ statusCode: 401, message: 'Invalid or expired token' })
  }
}
```

---

## Pages & Layouts Skill

### Page template with all features

```vue
<!-- pages/blog/[slug].vue -->
<script setup lang="ts">
// 1. Page meta (static)
definePageMeta({
  name: 'BlogPost',
  layout: 'blog',
  middleware: ['auth'],
})

// 2. Route
const route = useRoute()
const slug = computed(() => route.params.slug as string)

// 3. Data fetching
const { data: post, status } = await useFetch<Post>(`/api/posts/${slug.value}`, {
  key: `post-${slug.value}`,
})

// 4. Handle not found
if (!post.value) {
  throw createError({ statusCode: 404, message: 'Post not found' })
}

// 5. SEO
useSeoMeta({
  title: post.value.title,
  description: post.value.excerpt,
  ogTitle: post.value.title,
  ogDescription: post.value.excerpt,
  ogImage: post.value.coverImage,
  ogType: 'article',
  twitterCard: 'summary_large_image',
})
</script>

<template>
  <div v-if="status === 'pending'">
    <PostSkeleton />
  </div>
  <article v-else-if="post">
    <h1>{{ post.title }}</h1>
    <div v-html="post.renderedContent" />
  </article>
</template>
```

### Layout template

```vue
<!-- layouts/dashboard.vue -->
<script setup lang="ts">
const authStore = useAuthStore()
const { user } = storeToRefs(authStore)
</script>

<template>
  <div class="dashboard-layout">
    <AppSidebar />
    <main class="dashboard-main">
      <AppTopbar :user="user" />
      <div class="dashboard-content">
        <slot /> <!-- Pages render here -->
      </div>
    </main>
  </div>
</template>
```

---

## Middleware Skill

### Named middleware (opt-in per page)

```ts
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const { isAuthenticated, user } = storeToRefs(useAuthStore())

  if (!isAuthenticated.value) {
    return navigateTo({
      name: 'Login',
      query: { redirect: to.fullPath },
    })
  }

  // Role check
  if (to.meta.requiredRole && user.value?.role !== to.meta.requiredRole) {
    return abortNavigation(
      createError({ statusCode: 403, message: 'Forbidden' }),
    )
  }
})
```

### Global middleware (runs on every navigation)

```ts
// middleware/analytics.global.ts
export default defineNuxtRouteMiddleware((to) => {
  // Runs automatically on every route change
  if (import.meta.client) {
    trackPageView(to.fullPath)
  }
})
```

---

## Plugin Skill

```ts
// plugins/toast.client.ts — client-only
import Toast from 'vue-toastification'

export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.use(Toast, {
    position: 'top-right',
    timeout: 3000,
  })

  // Provide typed helper
  nuxtApp.provide('toast', {
    success: (msg: string) => useToast().success(msg),
    error: (msg: string) => useToast().error(msg),
    info: (msg: string) => useToast().info(msg),
  })
})

// Augment types
declare module '#app' {
  interface NuxtApp {
    $toast: {
      success(msg: string): void
      error(msg: string): void
      info(msg: string): void
    }
  }
}
```

---

## SEO Skill

### Global defaults in `app.vue`

```ts
// app.vue
useSeoMeta({
  titleTemplate: '%s | My Brand',
  description: 'Default site description for social sharing',
  ogSiteName: 'My Brand',
  ogImage: 'https://mysite.com/og-default.png',
  twitterCard: 'summary_large_image',
  twitterSite: '@mybrand',
})
```

### Dynamic SEO per page

```ts
// Computed SEO from fetched data
watchEffect(() => {
  if (product.value) {
    useSeoMeta({
      title: product.value.name,
      description: product.value.shortDescription,
      ogImage: product.value.images[0]?.url,
      ogType: 'product',
    })
  }
})
```

### Structured data (JSON-LD)

```ts
useHead({
  script: [
    {
      type: 'application/ld+json',
      innerHTML: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'Article',
        headline: post.value.title,
        author: { '@type': 'Person', name: post.value.author.name },
        datePublished: post.value.publishedAt,
      }),
    },
  ],
})
```

---

## Rendering Strategy Reference

| Use Case | Strategy | Config |
|---|---|---|
| Marketing pages | Full static | `prerender: true` |
| Blog / docs | ISR (hourly) | `isr: 3600` |
| Product pages | ISR (15 min) | `isr: 900` |
| Dashboard / app | Client-only SPA | `ssr: false` |
| Default (dynamic) | SSR per request | (default, no config needed) |
| API routes | No cache | `cache: false` |

---

## Common Anti-Patterns to Avoid

| ❌ Wrong | ✅ Correct |
|---|---|
| `import { useFetch } from '#app'` | Just use `useFetch` — it's auto-imported |
| `import { ref } from 'vue'` | Auto-imported — remove the import |
| `window.localStorage` in `<script setup>` | Wrap in `onMounted` or use `.client.ts` plugin |
| `process.env.SECRET` in component | Use `useRuntimeConfig().public.xxx` (public only) |
| `fetch('/api/users')` in `<script setup>` | Use `useFetch('/api/users')` for SSR |
| `onMounted(() => { fetch data })` for SSR | Use `useFetch` or `useAsyncData` instead |
| `<a href="/about">` for internal links | Use `<NuxtLink to="/about">` |
| `router.push()` before `navigateTo()` | Use `navigateTo()` — Nuxt-aware navigation |