<h1 align="center">base-cli</h1>
<p align="center">Configuration, login, update and logging utilities for TypeScript CLIs.</p>
<p align="center"><a href="../README.md">简体中文</a></p>

## What it does

`@nocoo/base-cli` is a reusable CLI library for configuration files, browser OAuth, package updates and logging. It does not ship a standalone application or hosted service.

## Features

- **ConfigManager**: generic configuration with development/production separation, synchronous/asynchronous APIs and mode-0600 files.
- **Login flow**: browser OAuth through a loopback callback server with reflected-HTML protection.
- **Update helpers**: detect Bun/pnpm/Yarn/npm and query the npm registry for versions.
- **Version utilities**: read package versions and compare semantic versions.
- **Browser**: open URLs on macOS, Windows and Linux.
- **Logging**: consola plus duration, size and date formatting.

## Usage

```bash
bun add @nocoo/base-cli
# Choose one
npm install @nocoo/base-cli
```

This package was renamed from `@nocoo/cli-base`; install `@nocoo/base-cli` and update your import paths.

### Configuration

```typescript
import { homedir } from "node:os";
import { join } from "node:path";
import { ConfigManager } from "@nocoo/base-cli";

type MyConfig = {
  token?: string;
  deviceId?: string;
}

const config = new ConfigManager<MyConfig>(
  join(homedir(), ".config", "my-cli"),
  false,
);
config.write({ token: "xxx" });
const token = config.get("token");
```

### Login

```typescript
import { performLogin, openBrowser } from "@nocoo/base-cli";

const result = await performLogin({
  openBrowser,
  onSaveToken: (token) => config.write({ token }),
  apiUrl: "https://my-saas.com",
  timeoutMs: 120_000,
});
```

### Update checks

```typescript
import { detectPackageManager, getLatestVersion, getUpdateCommand } from "@nocoo/base-cli";

const pm = detectPackageManager("@nocoo/my-cli");
const latest = await getLatestVersion("@nocoo/my-cli");
if (pm && latest) {
  console.log(getUpdateCommand(pm, "@nocoo/my-cli"));
}
```

## Development

Use Bun to install dependencies; CI pins Bun 1.3.14. Source lives in `src/`; TypeScript emits JavaScript and declarations to `dist/`.

```sh
bun install --frozen-lockfile
bun run typecheck
bun run lint
bun run build
```

The npm package publishes `dist/`. Typechecking alone does not generate a publishable artifact. See the maintenance guide for versioning and publication.

## Tests

```sh
bun run test
bun run test:coverage
```

Vitest covers configuration, version, update and logging utilities, including real loopback HTTP login callbacks. Configuration tests use temporary directories and never log into a real service.

## Stack

| Technology | Role |
| --- | --- |
| citty | CLI command framework |
| consola | Logging |
| picocolors | Terminal colors |
| yocto-spinner | Progress indicators |
| Vitest | Tests |

## Documentation

- [Public exports](../src/index.ts) and [configuration implementation](../src/config.ts).
- [OAuth login contract](../src/login.ts) and [update utilities](../src/update.ts).
- [Maintenance and publication](../AGENTS.md), [changelog](../CHANGELOG.md).

## License

[MIT](../LICENSE)
