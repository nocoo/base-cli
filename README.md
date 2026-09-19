<h1 align="center">base-cli</h1>
<p align="center">为 TypeScript CLI 提供配置、登录、更新与日志工具。</p>
<p align="center"><a href="docs/README.en.md">English</a></p>

## 这是什么

`@nocoo/base-cli` 是供 CLI 项目复用的基础库，提供配置文件、浏览器 OAuth、包更新和日志等通用能力。它不提供独立应用或托管服务。

## 功能

- **ConfigManager** — 泛型配置文件管理，支持 dev/prod 环境分离，sync/async API，0600 权限保护
- **Login Flow** — 浏览器 OAuth 登录流程，本地 loopback 回调服务器，XSS 防护
- **Update Helpers** — 自动检测 bun/pnpm/yarn/npm，查询 npm registry 最新版本
- **Version Utils** — 从 package.json 读取版本，语义化版本比较
- **Browser** — 跨平台打开浏览器（macOS/Windows/Linux）
- **Log** — consola 封装，formatDuration/formatSize/formatDate 格式化工具

## 使用

```bash
bun add @nocoo/base-cli
# 二选一
npm install @nocoo/base-cli
```

> **迁移提示**：本包由 `@nocoo/cli-base` 更名而来，旧包已 deprecate。请直接安装 `@nocoo/base-cli`，import 路径同步替换即可，API 完全兼容。

### 配置管理

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

### 登录流程

```typescript
import { performLogin, openBrowser } from "@nocoo/base-cli";

const result = await performLogin({
  openBrowser,
  onSaveToken: (token) => config.write({ token }),
  apiUrl: "https://my-saas.com",
  timeoutMs: 120_000,
});
```

### 更新检测

```typescript
import { detectPackageManager, getLatestVersion, getUpdateCommand } from "@nocoo/base-cli";

const pm = detectPackageManager("@nocoo/my-cli");
const latest = await getLatestVersion("@nocoo/my-cli");
if (pm && latest) {
  console.log(getUpdateCommand(pm, "@nocoo/my-cli"));
}
```

## 开发

使用 Bun 安装依赖；CI 固定 Bun 1.3.14。源码在 `src/`，TypeScript 构建输出 JavaScript 与声明文件到 `dist/`。

```sh
bun install --frozen-lockfile
bun run typecheck
bun run lint
bun run build
```

npm 包发布 `dist/`；只做类型检查不会生成可发布构件。版本和发布流程见维护说明。

## 测试

```sh
bun run test
bun run test:coverage
```

Vitest 覆盖配置、版本、更新与日志工具，以及实际 loopback HTTP 登录回调。配置使用临时目录；测试不登录真实服务。

## 技术栈

| 层 | 技术 |
|---|------|
| CLI 框架 | [citty](https://github.com/unjs/citty) |
| 日志 | [consola](https://github.com/unjs/consola) |
| 颜色 | [picocolors](https://github.com/alexeyraspopov/picocolors) |
| 进度 | [yocto-spinner](https://github.com/sindresorhus/yocto-spinner) |
| 测试 | [Vitest](https://vitest.dev) |

## 文档

- [公共导出](src/index.ts)与[配置实现](src/config.ts)。
- [OAuth 登录契约](src/login.ts)与[更新工具](src/update.ts)。
- [维护与发布说明](CLAUDE.md)、[变更记录](CHANGELOG.md)。

## 许可证

[MIT](LICENSE)
