# Migration Workflow from @alova/wormhole to worma

When migrating a project from `@alova/wormhole` to worma, follow this efficient flow:

1. **Create the worma config and migrate `alova.config` over**:
   ```bash
   npx wormajs init --template alovaGlobals
   ```
   This scaffolds `worma.config.{js,ts}` and migrates the existing `alova.config` into it. Then delete the old config file:
   ```bash
   rm alova.config.{js,ts}
   ```

[installation-config](https://worma.js.org/llms.mdx/docs/guide/installation-config.md) for more details.

2. **Swap the dependency to the latest version directly** — no need to look up a version number first:

```bash
pnpm remove @alova/wormhole
pnpm add wormajs -D        # installs latest, equivalent to @latest
```

3. **Follow the official migration guide** for the remaining config/code details — template plugin, package & import mapping.
   detail → [From wormhole migration](https://worma.js.org/llms.mdx/docs/migration/from-wormhole.md)
4. **Locate any remaining references** by searching the keyword `@alova/wormhole` across the repo (e.g. `package.json`, config, imports). Runtime API code under `output` is generated output and consumer code.
5. **Regenerate** with `worma gen` (add a `gen` script: `"gen": "worma gen"`) to verify the config loads and `output` regenerates.
