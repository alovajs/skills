---
name: alova-client
description: Best practices for using alova v3 in browser/client-side applications (Vue3, React, Svelte). Use this skill whenever the user asks about alova client-side usage, including setup, useRequest, useWatcher, useFetcher, caching, usePagination, useForm, useAutoRequest, actionDelegationMiddleware, or any alova/client imports. Also trigger when user mentions integrating alova with Vue/React/Svelte, managing request state, client-side cache strategies, or building paginated lists/forms with alova.
---

# Alova Client-Side Skill (v3)

> **Scope**: Browser environments — Vue3 / React / Svelte.  
> For server-side (Node/Bun/Deno), see `alova-server` skill.  
> For mini-programs (uni-app/Taro), see `alova-miniprogram` skill.

---

## How to Use This Skill

This skill is structured in two layers:

1. **This file** — Quick-reference index: what each API does and when to use it. Read this first.
2. **Official docs (fetch on demand)** — For full options, edge cases, or unfamiliar APIs, use `web_fetch` on the URL listed in each section to get the latest accurate information.

> **Always fetch the official doc before answering questions about specific API options or behaviors** — alova is actively developed and live docs are more reliable than training data.

---

## 1. Installation & Entry Point

```bash
npm install alova
```

**v3 package structure** (all in one package):
```js
import { createAlova } from 'alova';
import adapterFetch from 'alova/fetch';
import VueHook from 'alova/vue';          // or 'alova/react' / 'alova/svelte'
import { useRequest, useWatcher, useFetcher, usePagination, useForm, ... } from 'alova/client';
```

📄 Setup docs → `https://alova.js.org/tutorial/getting-started/quick-start/`

---

## 2. Core Concepts

### `createAlova` — Create the shared instance (once, export it)
Configure: `baseURL`, `statesHook`, `requestAdapter`, `beforeRequest`, `responded`.  
**v3**: `responded` (not `responsed`) takes an object `{ onSuccess, onError }`.

📄 → `https://alova.js.org/tutorial/getting-started/global-interceptor/`

### Method instances — API definitions (high-cohesion pattern)
Define each API as a factory function returning a Method instance. Keep request params, cache config, and response transforms co-located — not scattered across components.

📄 → `https://alova.js.org/tutorial/getting-started/method/`

---

## 3. Core Hooks

| Hook | When to use |
|---|---|
| `useRequest` | Fetch on mount, or trigger once on a user action (button click, form submit) |
| `useWatcher` | Re-fetch automatically when reactive state changes (search input, filter, tab, page) |
| `useFetcher` | Preload data silently in background, or refresh from outside the component that owns the data |

**Decision rules**:
- `useRequest` → pass a **Method instance** directly
- `useWatcher` → first arg must be a **factory function** `() => method(state.value)`; add `immediate: true` to also fetch on mount (v3 default is `false`); use `debounce` for high-frequency state changes
- `useFetcher` → best for hover-prefetch, post-navigation preload, or cross-component refresh

📄 useRequest → `https://alova.js.org/tutorial/client/strategy/use-request/`  
📄 useWatcher → `https://alova.js.org/tutorial/client/strategy/use-watcher/`  
📄 useFetcher → `https://alova.js.org/tutorial/client/strategy/use-fetcher/`

---

## 4. Cache Strategy

Alova has L1 (memory) and L2 (persistent/restore) layers, plus automatic request sharing (dedup).

| Need | Approach |
|---|---|
| Fast in-page access, resets on refresh | `cacheFor: ms` (memory, default) |
| Survive page refresh / offline-first | `cacheFor: { mode: 'restore', expire: ms }` |
| Always re-fetch | `cacheFor: 0` |
| Auto-invalidate after a mutation | `hitSource` on GET + `name` on mutation Method |
| Manual invalidate | `await invalidateCache(method)` |
| Optimistic update (same page, component mounted) | `updateState(method, updater)` |
| Optimistic update (cross-page or component unmounted) | `await setCache(method, updater)` |

**Key rule**: prefer `hitSource` auto-invalidation — it requires zero imperative code and decouples components.  
**v3 note**: `invalidateCache`, `setCache`, `queryCache` are all **async** — always `await` them.

📄 → `https://alova.js.org/tutorial/client/cache/`

---

## 5. Business Strategy Hooks (`alova/client`)

Use these instead of hand-rolling common request patterns.

| Scenario | Hook | Key capability |
|---|---|---|
| Paginated list / infinite scroll | `usePagination` | Auto page management, preload next/prev, optimistic insert/remove/replace |
| Form submit (any complexity) | `useForm` | Draft persistence, multi-step state sharing, auto-reset |
| Polling / focus / reconnect refresh | `useAutoRequest` | Configurable triggers, throttle |
| Captcha send + countdown | `useCaptcha` | Cooldown timer built-in |
| Cross-component request trigger | `actionDelegationMiddleware` + `accessAction` | No prop-drilling or global store |
| Chained dependent requests | `useSerialRequest` / `useSerialWatcher` | Each step receives previous result |
| Retry with exponential backoff | `useRetriableRequest` | Configurable attempts + jitter |
| File upload with progress | `useUploader` | Concurrent limit, progress events |
| Server-Sent Events | `useSSE` | Reactive `data` + `readyState` |

📄 Fetch the specific page for full options + examples:

| Hook | Doc URL |
|---|---|
| usePagination | `https://alova.js.org/tutorial/client/strategy/use-pagination/` |
| useForm | `https://alova.js.org/tutorial/client/strategy/use-form/` |
| useAutoRequest | `https://alova.js.org/tutorial/client/strategy/use-auto-request/` |
| useCaptcha | `https://alova.js.org/tutorial/client/strategy/use-captcha/` |
| actionDelegationMiddleware | `https://alova.js.org/tutorial/client/strategy/action-delegation-middleware/` |
| useRetriableRequest | `https://alova.js.org/tutorial/client/strategy/use-retriable-request/` |
| useSSE | `https://alova.js.org/tutorial/client/strategy/use-sse/` |
| useUploader | `https://alova.js.org/tutorial/client/strategy/use-uploader/` |

---

## 6. Common Pitfalls

| Pitfall | Fix |
|---|---|
| `useWatcher` first arg is a Method instance | Always wrap: `() => method(state.value)` |
| `updateState` silently does nothing | Only works while owning component is mounted; use `setCache` otherwise |
| Cache ops called synchronously in v3 | `await invalidateCache / setCache / queryCache` |
| `responded` misspelled as `responsed` | v3 uses `responded` |
| `useWatcher` doesn't fetch on mount | Set `immediate: true` |

---

## 7. TypeScript

Annotate the response shape on the Method instance — hooks infer from it automatically:
```ts
const getUser = (id: number) => alovaInstance.Get<User>(`/users/${id}`);
const { data } = useRequest(getUser(1)); // data: Ref<User>
```

📄 → `https://alova.js.org/tutorial/client/typescript/`
