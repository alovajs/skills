---
name: alova-server
description: Best practices for using alova v3 in server-side environments (Node.js, Bun, Deno) and SSR frameworks (Next.js, Nuxt3, SvelteKit). Use this skill whenever the user asks about alova on the server side, including BFF layers, API gateways, microservice forwarding, server hooks (retry, rateLimit, atomize), token authentication, distributed caching with Redis, SSR data fetching, or any alova/server imports. Also trigger when the user mentions using alova in Node.js/Bun/Deno, building request forwarding layers, or handling alova in SSR contexts like Next.js app router or Nuxt3 useAsyncData.
---

# Alova Server-Side Skill (v3)

> **Scope**: Node.js / Bun / Deno runtimes + SSR frameworks (Next.js, Nuxt3, SvelteKit).  
> For browser/component usage, see `alova-client` skill.

---

## How to Use This Skill

1. **This file** — Index and decision layer. Read this first to orient yourself.
2. **Official docs (fetch on demand)** — For full options or edge cases, use `web_fetch` on the URLs listed in each section.

> **Always fetch the official doc before answering specific API questions** — server-side features in alova are newer and evolve quickly.

---

## 1. Installation & Key Difference from Client

```bash
npm install alova
```

```js
import { createAlova } from 'alova';
import adapterFetch from 'alova/fetch';
import { retry, RateLimiter, atomize } from 'alova/server';
```

**Core difference**: No `statesHook` needed. Server-side alova uses plain `async/await`:
```js
const user = await alovaInstance.Get('/users/1').send();
```

📄 → `https://alova.js.org/tutorial/server/strategy/`

---

## 2. `createAlova` for Server

```js
export const alovaInstance = createAlova({
  baseURL: 'https://internal-api.example.com',
  requestAdapter: adapterFetch(),      // No statesHook
  beforeRequest(method) {
    method.config.headers['X-Service-Token'] = process.env.SERVICE_TOKEN;
  },
  responded: {
    onSuccess: async (response) => {
      const json = await response.json();
      if (!json.success) throw new Error(json.message);
      return json.data;
    },
    onError: (error) => { throw error; }
  }
});
```

📄 → `https://alova.js.org/tutorial/getting-started/global-interceptor/`

---

## 3. Server Hooks — Decision Guide

Server hooks wrap a Method instance and return a new hooked Method. They are **composable** and all **support cluster mode**.

| Need | Hook |
|---|---|
| Retry failed requests with backoff | `retry` |
| Rate-limit outgoing requests | `RateLimiter` |
| Only one process initiates at a time (cluster) | `atomize` |
| Silent token refresh + request queuing | `createServerTokenAuthentication` |

**Composing hooks**:
```js
// Innermost wraps first: limit → then retry the limited method
const result = await retry(
  limiter.limit(alovaInstance.Post('/api/order', data)),
  { retry: 3 }
).send();
```

📄 → `https://alova.js.org/tutorial/server/strategy/`

---

## 4. Server Hooks

### `retry`
```js
import { retry } from 'alova/server';
const result = await retry(alovaInstance.Get('/api/user'), {
  retry: 3,
  backoff: { delay: 1000, multiplier: 2, endQuiver: 0.3 }
}).send();
```
📄 → `https://alova.js.org/tutorial/server/strategy/retry/`

### `RateLimiter`
```js
import { RateLimiter } from 'alova/server';
const limiter = new RateLimiter({ points: 10, duration: 1000 });
const result = await limiter.limit(alovaInstance.Get('/third-party/api')).send();
```
For cluster mode, supply a Redis storage adapter so all processes share the same counter.

📄 → `https://alova.js.org/tutorial/server/strategy/rate-limit/`

### `atomize`
Ensures only one process initiates a request at a time — critical for token refresh or one-time resource init across Node.js workers.
```js
import { atomize } from 'alova/server';
const token = await atomize(alovaInstance.Get('/api/access_token')).send();
```
Requires a shared storage adapter (Redis) to coordinate across processes.

📄 → `https://alova.js.org/tutorial/server/strategy/atomize/`

---

## 5. Token Authentication

`createServerTokenAuthentication` handles the full token lifecycle in one place: attach, detect expiry, silent refresh, queue in-flight requests, and guest bypass.

```js
import { createServerTokenAuthentication } from 'alova/client';

const { onAuthRequired, onResponseRefreshToken } = createServerTokenAuthentication({
  refreshTokenOnSuccess: {
    isExpired: (response) => response.status === 401,
    handler: async () => {
      const { token } = await refreshToken().send();
      saveToken(token);
    }
  },
  assignToken: (method) => {
    method.config.headers.Authorization = `Bearer ${getToken()}`;
  }
});

export const alovaInstance = createAlova({
  beforeRequest: onAuthRequired(),
  responded: onResponseRefreshToken({ onSuccess: async (res) => res.json() })
});
```

**Guest requests** (skip token): set `method.meta = { authRole: null }`.

📄 → `https://alova.js.org/tutorial/client/strategy/token-authentication/`

---

## 6. Distributed Caching

| Scenario | Storage adapter |
|---|---|
| Single-process | Default in-memory |
| Multi-process cluster | `@alova/storage-redis` |
| Single-machine cluster | `@alova/storage-file` |

**Multi-level cache** (L1 memory + L2 Redis):
```js
createAlova({
  cacheFor: {
    GET: {
      mode: 'restore',
      expire: ({ mode }) =>
        mode === 'memory' ? 5 * 60 * 1000 : 24 * 60 * 60 * 1000,
    }
  }
});
```

📄 → `https://alova.js.org/tutorial/server/strategy/`

---

## 7. SSR Frameworks

**Rule**: Never use `useRequest`/`useWatcher` in server data loaders — call `.send()` directly. Use hooks normally once running in the browser.

> **Cache pitfall**: Server and client caches are not shared. If a GET is cached server-side but not client-side, hydration will trigger a loading flash. Fix: pass server data as `initialData` to `useRequest`, or disable cache server-side.

### Next.js (App Router)
```js
// async Server Component — direct .send()
export default async function UsersPage() {
  const users = await getUserList().send();
  return <UserList data={users} />;
}
```

### Nuxt3
```vue
<script setup>
const { data } = useAsyncData('users', () => getUserList().send());
</script>
```

### SvelteKit
```js
// +page.server.js
export async function load() {
  return { users: await getUserList().send() };
}
```

📄 → `https://alova.js.org/tutorial/server/ssr/`

---

## 8. BFF / API Gateway Pattern

Use `async_hooks` to capture per-request context and forward user headers to upstream services:

```js
import { AsyncLocalStorage } from 'async_hooks';
const requestContext = new AsyncLocalStorage();

export const alovaInstance = createAlova({
  beforeRequest(method) {
    const ctx = requestContext.getStore();
    if (ctx?.userId) method.config.headers['X-User-Id'] = ctx.userId;
  }
});

// Express/Fastify middleware
app.use((req, res, next) => {
  requestContext.run({ userId: req.user?.id }, next);
});
```

---

## 9. Common Pitfalls

| Pitfall | Fix |
|---|---|
| Using `useRequest` in a server data loader | Call `.send()` directly in the loader |
| Including `statesHook` in server alova instance | Omit it — not needed server-side |
| SSR hydration flash (server cached, client misses) | Pass server data via `initialData` or disable cache server-side |
| `RateLimiter` state not shared across workers | Add a Redis storage adapter to `RateLimiter` options |
| `atomize` not actually atomic in cluster | Requires a shared storage adapter (Redis) to coordinate processes |
