# Vue 3 Skill

## Description
Use this skill when the task involves **creating, editing, or reviewing Vue 3 components, composables, Pinia stores, or Vue Router configuration**. This skill provides deep expertise in Vue 3 Composition API patterns, component architecture, state management, and performance optimization.

Trigger phrases: "create a component", "write a composable", "set up Pinia", "build a form", "add Vue Router", "make it reactive", "refactor to Composition API", "build a modal", "create a reusable component", "add a store", "should I use Pinia or TanStack Query", "set up TanStack Query", "axios vs tanstack", "when to use Pinia", "state management in Vue", "cache API data", "data fetching setup".

---

## How to Use This Skill

When working on Vue 3 tasks, follow this workflow:

1. **Understand the component's responsibility** — one component = one job
2. **Identify what state is needed** — local `ref`/`reactive`, composable, Pinia (client state), or TanStack Query (server state) — see State Management Decision Guide below
3. **Identify what the parent needs to know** — design the `emit` interface first
4. **Write `<script setup>` before the template** — logic drives structure
5. **Write the template to match the logic** — not the other way around
6. **Add scoped styles last** — no global leakage

---

## Component Skeleton

Every Vue 3 component must follow this exact structure:

```vue
<script setup lang="ts">
// 1. defineOptions (name for DevTools)
defineOptions({ name: 'ComponentName' })

// 2. Props interface + defineProps
interface Props {
  title: string
  count?: number
  isDisabled?: boolean
}
const props = withDefaults(defineProps<Props>(), {
  count: 0,
  isDisabled: false,
})

// 3. Emits
const emit = defineEmits<{
  submit: [value: string]
  cancel: []
  update: [field: string, value: unknown]
}>()

// 4. Injected dependencies (useRoute, useRouter, stores)
const router = useRouter()
const authStore = useAuthStore()

// 5. Local reactive state
const isOpen = ref(false)
const inputValue = ref('')

// 6. Computed values
const isValid = computed(() => inputValue.value.trim().length > 0)
const displayTitle = computed(() => props.title.toUpperCase())

// 7. Composables
const { user, isLoading } = useUser(props.userId)

// 8. Methods / handlers
function handleSubmit() {
  if (!isValid.value) return
  emit('submit', inputValue.value)
}

// 9. Lifecycle hooks
onMounted(() => {
  // DOM is available here
})

onUnmounted(() => {
  // cleanup here
})
</script>

<template>
  <div class="component-root">
    <!-- template content -->
  </div>
</template>

<style scoped>
.component-root {
  /* scoped styles only */
}
</style>
```

---

## Composable Skill

### When to extract a composable
- Logic is used in 2+ components → extract immediately
- Logic has its own loading/error state → extract always
- Logic involves async operations → extract always
- Logic involves event listeners or timers → extract and clean up

### Composable template

```ts
// composables/useResourceName.ts

import type { MaybeRef } from 'vue'

interface ResourceNameOptions {
  immediate?: boolean
  onError?: (err: Error) => void
}

export function useResourceName(
  id: MaybeRef<string>,
  options: ResourceNameOptions = {},
) {
  const { immediate = true, onError } = options

  // State
  const data = ref<ResourceType | null>(null)
  const isLoading = ref(false)
  const error = ref<Error | null>(null)

  // Core async function
  async function fetch() {
    if (!toValue(id)) return

    isLoading.value = true
    error.value = null

    try {
      data.value = await apiClient.get<ResourceType>(`/resource/${toValue(id)}`)
    } catch (err) {
      const e = err instanceof Error ? err : new Error(String(err))
      error.value = e
      onError?.(e)
    } finally {
      isLoading.value = false
    }
  }

  async function update(payload: Partial<ResourceType>) {
    try {
      data.value = await apiClient.patch(`/resource/${toValue(id)}`, payload)
    } catch (err) {
      error.value = err instanceof Error ? err : new Error(String(err))
    }
  }

  // Reactive refetch when id changes
  watch(() => toValue(id), fetch, { immediate })

  return {
    data: readonly(data),
    isLoading: readonly(isLoading),
    error: readonly(error),
    fetch,
    update,
  }
}
```

### Key composable rules
- Return `readonly()` wrapped refs to prevent accidental mutation from outside
- Accept `MaybeRef<T>` for params so callers can pass either a `ref` or a raw value
- Use `toValue()` (Vue 3.3+) inside the composable to unwrap either
- Always clean up event listeners, timers, subscriptions inside `onUnmounted` or `watchEffect` cleanup

---

## Pinia Store Skill

### Store template (always use Setup Store style)

```ts
// stores/useExampleStore.ts
import { defineStore } from 'pinia'

interface ExampleState {
  items: Item[]
  selectedId: string | null
  filter: string
}

export const useExampleStore = defineStore('example', () => {
  // --- State ---
  const items = ref<Item[]>([])
  const selectedId = ref<string | null>(null)
  const filter = ref('')

  // --- Getters (computed) ---
  const selectedItem = computed(() =>
    items.value.find(i => i.id === selectedId.value) ?? null,
  )

  const filteredItems = computed(() =>
    filter.value
      ? items.value.filter(i =>
          i.name.toLowerCase().includes(filter.value.toLowerCase()),
        )
      : items.value,
  )

  // --- Actions ---
  async function loadItems() {
    try {
      items.value = await $fetch<Item[]>('/api/items')
    } catch (err) {
      console.error('[ExampleStore] loadItems failed:', err)
    }
  }

  async function createItem(payload: CreateItemPayload) {
    const newItem = await $fetch<Item>('/api/items', {
      method: 'POST',
      body: payload,
    })
    items.value.push(newItem)
    return newItem
  }

  function selectItem(id: string) {
    selectedId.value = id
  }

  function setFilter(value: string) {
    filter.value = value
  }

  // --- Reset ---
  function $reset() {
    items.value = []
    selectedId.value = null
    filter.value = ''
  }

  return {
    // State (readonly to enforce actions)
    items: readonly(items),
    selectedId: readonly(selectedId),
    filter: readonly(filter),
    // Getters
    selectedItem,
    filteredItems,
    // Actions
    loadItems,
    createItem,
    selectItem,
    setFilter,
    $reset,
  }
})
```

### Using the store in a component

```ts
// In component
const store = useExampleStore()

// ✅ Use storeToRefs for reactive state
const { items, selectedItem, filteredItems } = storeToRefs(store)

// ✅ Destructure actions directly (not reactive, they're functions)
const { loadItems, createItem, selectItem } = store

onMounted(() => loadItems())
```

---

## State Management Decision Guide

This is one of the most common points of confusion in modern Vue development. The short answer: **Pinia and TanStack Query are not competitors — they solve different problems and are often used together in the same project.**

### The Core Mental Model: Who Owns the Data?

```
Is this data from a server / API / database?
  └─ YES → TanStack Query owns it
      Examples: user profiles, product lists, orders, search results

Is this data purely inside your app (no server round-trip needed)?
  └─ YES → Pinia owns it
      Examples: sidebar open/closed, dark mode, current step in a wizard, auth token
```

### When to Use TanStack Query

Use TanStack Query for any **server state** — data that lives on an external server and needs to be synchronized with your UI.

**What it handles automatically so you don't have to:**
- `isLoading`, `isError`, `data` states — no manual `ref` boilerplate
- **Caching** — navigate away and back, data is served from cache instantly
- **Deduplication** — two components requesting the same data = one network request
- **Background refetching** — stale data is refreshed when the tab regains focus
- **Pagination & infinite scroll** — built-in primitives
- **Memory management** — garbage-collects data no longer used by any component

```ts
// ❌ Old way — raw Axios in onMounted (every dev writes this boilerplate)
const users = ref<User[]>([])
const isLoading = ref(false)
const error = ref<Error | null>(null)

onMounted(async () => {
  isLoading.value = true
  try {
    const res = await axios.get('/api/users')
    users.value = res.data
  } catch (err) {
    error.value = err as Error
  } finally {
    isLoading.value = false
  }
})
// 👆 No caching. Refetches on every mount. Duplicates if two components need same data.

// ✅ TanStack Query way — all of the above, plus caching + dedup + background sync
import { useQuery } from '@tanstack/vue-query'

const { data: users, isLoading, isError, error } = useQuery({
  queryKey: ['users'],
  queryFn: () => axios.get<User[]>('/api/users').then(res => res.data),
  staleTime: 1000 * 60 * 5, // treat as fresh for 5 minutes
})
```

### When to Use Pinia

Use Pinia for **client state** — data that exists purely inside your frontend and doesn't need a server round-trip.

| Good Pinia use cases | Why not TanStack Query |
|---|---|
| Dark mode / theme preference | No server — it's a UI toggle |
| Sidebar collapsed / expanded | Local UI state, no async |
| Current authenticated user's display info | Populated once from auth, then referenced everywhere |
| Multi-step form data across route changes | Temporary local data, not persisted server-side |
| Selected filters that affect multiple views | Shared UI state between components |
| Shopping cart (before checkout) | Local until user submits |

```ts
// stores/useUIStore.ts — perfect Pinia use case
export const useUIStore = defineStore('ui', () => {
  const isDarkMode = ref(false)
  const isSidebarOpen = ref(true)
  const activeLocale = ref<'en' | 'hi' | 'ur'>('en')

  function toggleDarkMode() { isDarkMode.value = !isDarkMode.value }
  function toggleSidebar() { isSidebarOpen.value = !isSidebarOpen.value }
  function setLocale(locale: typeof activeLocale.value) { activeLocale.value = locale }

  return { isDarkMode, isSidebarOpen, activeLocale, toggleDarkMode, toggleSidebar, setLocale }
})
```

### Axios vs TanStack Query — They're Not Alternatives

This is a key misconception. Think of them as different layers:

```
┌─────────────────────────────────────────┐
│         TanStack Query                  │  ← "WHEN": caching, retries, loading state,
│   manages the lifecycle of data         │     deduplication, background sync
├─────────────────────────────────────────┤
│              Axios                      │  ← "HOW": makes the actual HTTP call,
│   makes the actual HTTP request         │     handles headers, interceptors, auth tokens
└─────────────────────────────────────────┘
```

**The professional setup is to use Axios *inside* TanStack Query's `queryFn`:**

```ts
// lib/axios.ts — global Axios instance with auth interceptor
import axios from 'axios'

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10_000,
  headers: { 'Content-Type': 'application/json' },
})

// Auth interceptor — attach token to every request automatically
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('access-token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

// Response interceptor — handle 401 globally
apiClient.interceptors.response.use(
  res => res,
  async (error) => {
    if (error.response?.status === 401) {
      await useAuthStore().logout()
    }
    return Promise.reject(error)
  },
)
```

```ts
// composables/useUsers.ts — TanStack Query + Axios together
import { useQuery, useMutation, useQueryClient } from '@tanstack/vue-query'
import { apiClient } from '@/lib/axios'

// Query (GET)
export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const { data } = await apiClient.get<User[]>('/users')
      return data
    },
    staleTime: 1000 * 60 * 2, // 2 minutes
  })
}

// Mutation (POST / PATCH / DELETE) + cache invalidation
export function useCreateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (payload: CreateUserPayload) =>
      apiClient.post<User>('/users', payload).then(res => res.data),

    // After creating a user, invalidate the users list so it refetches
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] })
    },
  })
}
```

```ts
// In a component — clean, no boilerplate
const { data: users, isLoading, isError } = useUsers()
const { mutate: createUser, isPending } = useCreateUser()

function handleSubmit(form: CreateUserPayload) {
  createUser(form)
}
```

### Decision Summary

| Scenario | Tool |
|---|---|
| Fetch a list from `/api/users` | TanStack Query |
| Fetch a single record by ID | TanStack Query |
| Paginated table / infinite scroll | TanStack Query |
| POST / PATCH / DELETE with cache update | TanStack Query `useMutation` |
| Dark mode toggle | Pinia |
| Auth user display info (navbar, avatar) | Pinia |
| Multi-step wizard form state | Pinia |
| Shared filters across multiple views | Pinia |
| Global HTTP headers / auth tokens | Axios interceptors |
| Simple one-off script / utility | Raw `fetch` or Axios directly |

### ⚠️ Key Rule: Don't Bridge the Two

Never copy data from TanStack Query into a Pinia store. This creates two sources of truth that will get out of sync.

```ts
// ❌ Anti-pattern — bridging TanStack Query into Pinia
const { data: users } = useQuery({ queryKey: ['users'], queryFn: fetchUsers })
watch(users, (val) => { userStore.setUsers(val) }) // Now you have two copies

// ✅ Correct — components consume TanStack Query directly
const { data: users } = useUsers() // that's it
```

### Setup: Installing TanStack Query in Vue 3

```ts
// main.ts
import { VueQueryPlugin, QueryClient } from '@tanstack/vue-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,     // 1 minute default freshness
      retry: 2,                  // retry failed requests twice
      refetchOnWindowFocus: true, // refresh when tab regains focus
    },
  },
})

app.use(VueQueryPlugin, { queryClient })
```

---

## Vue Router Skill

### Typed route definitions

```ts
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'Home',
      component: () => import('@/pages/HomePage.vue'),
    },
    {
      path: '/users/:id',
      name: 'UserDetail',
      component: () => import('@/pages/UserDetailPage.vue'),
      // Route-level guard
      beforeEnter: (to) => {
        if (!to.params.id) return { name: 'Home' }
      },
    },
    {
      path: '/dashboard',
      name: 'Dashboard',
      component: () => import('@/layouts/DashboardLayout.vue'),
      meta: { requiresAuth: true },
      children: [
        {
          path: '',
          name: 'DashboardHome',
          component: () => import('@/pages/dashboard/DashboardHome.vue'),
        },
      ],
    },
    {
      path: '/:pathMatch(.*)*',
      name: 'NotFound',
      component: () => import('@/pages/NotFoundPage.vue'),
    },
  ],
})

// Global auth guard
router.beforeEach((to) => {
  const authStore = useAuthStore()
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    return { name: 'Login', query: { redirect: to.fullPath } }
  }
})

export default router
```

### Navigating in components

```ts
const router = useRouter()
const route = useRoute()

// Typed param access
const userId = computed(() => route.params.id as string)

// Navigate with named route
await router.push({ name: 'UserDetail', params: { id: '123' } })

// Navigate with query
await router.push({ name: 'Search', query: { q: searchTerm.value } })

// Go back
router.back()
```

---

## Reactivity Patterns Cheat Sheet

| Pattern | When to use |
|---|---|
| `ref<T>(value)` | Primitives, arrays, objects you'll reassign |
| `reactive({})` | Objects you won't destructure or reassign |
| `computed(() => ...)` | Derived values — cached until deps change |
| `watch(source, cb)` | Side effects when reactive data changes |
| `watchEffect(cb)` | Side effects that auto-track dependencies |
| `shallowRef()` | Large objects — only top-level reactivity needed |
| `toRefs(reactive)` | Destructure reactive without losing reactivity |
| `toValue(maybeRef)` | Unwrap ref or raw value inside composables |
| `readonly(ref)` | Expose state from composable/store without allowing mutation |
| `markRaw(obj)` | Non-reactive objects (chart instances, maps, sockets) |

---

## Common Component Patterns

### Async component with loading state
```vue
<script setup lang="ts">
const HeavyChart = defineAsyncComponent({
  loader: () => import('./HeavyChart.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorMessage,
  delay: 200,
  timeout: 5000,
})
</script>
```

### v-model on custom component
```vue
<!-- Parent -->
<MyInput v-model="email" />

<!-- MyInput.vue -->
<script setup lang="ts">
const model = defineModel<string>({ required: true })
</script>
<template>
  <input :value="model" @input="model = $event.target.value" />
</template>
```

### Provide / Inject (typed)
```ts
// In parent
const ThemeKey: InjectionKey<Ref<'light' | 'dark'>> = Symbol('theme')
provide(ThemeKey, ref('light'))

// In child
const theme = inject(ThemeKey) // typed as Ref<'light' | 'dark'> | undefined
```

### Teleport for modals
```vue
<Teleport to="body">
  <div v-if="isModalOpen" class="modal-overlay">
    <div class="modal">
      <slot />
    </div>
  </div>
</Teleport>
```

---

## Performance Checklist

Before completing any component, verify:
- [ ] `v-for` always has a stable `:key` (not index)
- [ ] `v-if` and `v-for` are never on the same element
- [ ] Heavy computations are in `computed()`, not template expressions
- [ ] Large non-reactive objects are wrapped in `markRaw()`
- [ ] Heavy components use `defineAsyncComponent()`
- [ ] Long lists (100+ items) use virtual scrolling
- [ ] Event listeners registered in `onMounted` are removed in `onUnmounted`