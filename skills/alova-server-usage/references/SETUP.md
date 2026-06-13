# Setup Guide

## Installation

```bash
npm install alova
```

## Create Alova Instance

```ts
function createAlova(options?: AlovaOptions): Alova;
```

the following is `AlovaOptions` schame.

| Name           | Type                        | Description                                                                                                                                |
| -------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| requestAdapter | object                      | Request adapter, required, [For Detail](https://alova.js.org/tutorial/advanced/custom/http-adapter.md)                                        |
| id             | string \| number            | Alova instance id, optional, [For Detail](https://alova.js.org/tutorial/cache/mode.md#set-alova-id)                                           |
| baseURL        | string                      | Base path, optional, default is empty, [For Detail](https://alova.js.org/tutorial/getting-started/basic/alova.md)                             |
| timeout        | number                      | Timeout, default is no timeout, [For Detail](https://alova.js.org/tutorial/getting-started/basic/alova.md)                                    |
| cacheFor       | object                      | Local cache configuration, default GET has 5000ms cache, [For Detail](https://alova.js.org/tutorial/cache/mode.md)                            |
| l1Cache        | object                      | Level1 cache adapter [For Detail](https://alova.js.org/tutorial/cache/mode.md)                                                                |
| l2Cache        | object                      | Level2 cache adapter, [For Detail](https://alova.js.org/tutorial/cache/mode.md)                                                               |
| beforeRequest  | function                    | Before request hook, [For Detail](https://alova.js.org/tutorial/getting-started/basic/global-interceptor.md)                                  |
| responded      | object \| function          | Request response hook, [For Detail](https://alova.js.org/tutorial/getting-started/basic/global-interceptor.md)                                |
| shareRequest   | boolean                     | Share request, [For Detail](https://alova.js.org/tutorial/getting-started/basic/alova.md)                                                     |
| cacheLogger    | boolean \| null \| function | Cache log, [For Detail](https://alova.js.org/tutorial/advanced/in-depth/cache-logger.md)                                                      |
| snapshots      | number                      | method The number of snapshots is limited, the default is 1000, [For Detail](https://alova.js.org/tutorial/client/in-depth/method-matcher.md) |

### Example

```js
import { createAlova } from 'alova';
import adapterFetch from 'alova/fetch';

export const alovaInstance = createAlova({
  baseURL: 'https://api.example.com',
  requestAdapter: adapterFetch(),
  beforeRequest(method) {
    method.config.headers.Authorization = 'Bearer token';
  },
  responded: {
    onSuccess: async (response) => response.json(),
    onError: (error) => console.error(error),
    onComplete: () => console.log('complete'),
  },
});
```

## Request Adapters

| Adapter | Import                 | Extra Install          | Use Case                    |
| ------- | ---------------------- | ---------------------- | --------------------------- |
| Fetch   | `alova/fetch`          | -                      | Browser, React Native, Expo |
| XHR     | `@alova/adapter-xhr`   | `@alova/adapter-xhr`   | Legacy browsers             |
| Axios   | `@alova/adapter-axios` | `@alova/adapter-axios` | Axios compatibility         |
