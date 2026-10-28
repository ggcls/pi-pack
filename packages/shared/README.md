# @pi-pack/shared

Shared TypeScript utilities for Pi extensions. This is a library, not a Pi
extension: it does not register event handlers or UI components.

## ESM imports

Import only the modules you need:

```ts
import { Events } from "@pi-pack/shared/events";
import { errorMessage } from "@pi-pack/shared/errors";
import { readOptionalFile } from "@pi-pack/shared/files";
import { booleanOption, isNil, isObject, unknownKeys } from "@pi-pack/shared/validation";
import { createConfigLoader, parseConfig } from "@pi-pack/shared/config";
```

Each subpath has explicit TypeScript and ESM import exports. There is no root
barrel, CommonJS entry point, or build step. Pi loads the TypeScript source; native
Node.js execution requires TypeScript stripping support. All functions are arrow
functions, and imports used only for types are erased at runtime.

The events, errors, files, and validation modules have no third-party runtime
imports. Configuration parsing uses `jsonc-parser` through Node's CommonJS/ESM
interop; it supports comments, trailing commas, and parse error locations. We do
not rely on private dependency paths or replace the parser with a regex.

## Configuration

```ts
import { getAgentDir } from "@earendil-works/pi-coding-agent";
import { createConfigLoader } from "@pi-pack/shared/config";
import { booleanOption } from "@pi-pack/shared/validation";

const loadConfig = createConfigLoader({
  name: "example",
  knownKeys: ["enabled"],
  defaults: () => ({ enabled: true }),
  validate: (value) => ({
    enabled: booleanOption(value.enabled, "enabled", true),
  }),
});

const config = await loadConfig(cwd, {
  agentDir: getAgentDir(),
  projectTrusted: false,
  onWarning: (message) => console.warn(message),
});
```

- Reads `<agentDir>/<name>.jsonc` by default.
- When `projectTrusted` is true, tries `<cwd>/.pi/<name>.jsonc` first. The first
  existing file wins; configurations are not merged.
- An untrusted project's configuration is never read.
- Missing files fall back to the next candidate, then to fresh defaults.
- Invalid or unreadable files fail with their path and original error cause.
- Unknown keys produce warnings but are not discarded. An `onWarning` callback
  takes precedence over `ui.notify`.
- Files are re-read on each call, so edits are not hidden by a configuration cache.

Provide `agentDir` to avoid any runtime import of the Pi SDK by this library. When
omitted, the loader lazily imports Pi and calls its `getAgentDir()` implementation,
preserving Pi's directory and environment-variable behavior.

`configPaths(name, cwd, agentDir)` requires an explicit agent directory and does
not load Pi. `parseConfig(source)` is synchronous and also does not load Pi.

Known keys are indexed in a `Set` once per loader. `unknownKeys` and
`warnUnknownConfig` also accept reusable sets, avoiding repeated array scans.
No parsed configurations, file contents, or user callbacks are cached globally.

## Development

From the workspace root:

```sh
pnpm --filter @pi-pack/shared test
pnpm --filter @pi-pack/shared test:watch
pnpm typecheck
```

Tests cover parsing, validation, file access, project trust, configuration
precedence, warnings, and native Node.js ESM import boundaries.

## License

[MIT](LICENSE)
