# Alova Agent Skills

A collection of AI agent skills for [alova](https://alova.js.org/). designed to enhance the capabilities of AI agents by providing a variety of functionalities. These skills can be easily integrated into your Alova Agent projects to enable features such as natural language processing, data analysis, and more. These skills enhance AI coding assistants with expert knowledge for alova integration and best practices.

## Included Skills

### alova-client-usage

Best practices for alova v3 in browser/client-side/SSR applications.

```bash
npx skills add alovajs/skills --skill alova-client-usage
```

### alova-server-usage

Best practices for alova v3 in server-side environments.

```bash
npx skills add alovajs/skills --skill alova-server-usage
```

### alova-wormhole-usage

Guide for integrating alova with OpenAPI/Swagger specs via `@alova/wormhole`.

```bash
npx skills add alovajs/skills --skill alova-wormhole-usage
```

### For AI Agent Users

Skills are automatically triggered when:

| Skill                | Trigger Keywords                                                                                |
| -------------------- | ----------------------------------------------------------------------------------------------- |
| alova-client-usage   | API requests, fetch data, alova client setup, `alova/client` imports, pagination, forms         |
| alova-server-usage   | Server-side requests, BFF, API gateway, Node.js/Bun/Deno, `alova/server` imports, Redis caching |
| alova-wormhole-usage | OpenAPI, Swagger, `@alova/wormhole`, API code generation, `alova gen`, `alova init`             |

## License

MIT
