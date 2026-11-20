# base-cli

Shared CLI infrastructure library (`@nocoo/base-cli`) for config management, OAuth loopback login, update checks, version utilities, browser opening, and logging.
Docs: [README.md](README.md) · [CHANGELOG.md](CHANGELOG.md) · [Retrospective](Retrospective.md)

## Commands

Run from root with Bun (`1.3.14` in CI) and Node (`26.12.1` in CI). Strict TypeScript 7 (`tsconfig.json` excludes `**/*.test.ts`), Biome (`biome.json`), and Vitest with V8 coverage (`vitest.config.ts`). Layout: `src/` modules with colocated `*.test.ts` plus `scripts/release.ts`. Gitleaks (`.gitleaks.toml`) and OSV Scanner (`osv-scanner.toml`) are required for security checks; tests need no production credentials.

```sh
bun install --frozen-lockfile
bun run typecheck          # S1 (excludes **/*.test.ts)
bun run lint               # S2: tsc --noEmit + biome check --error-on-warnings
bun run build              # B1: emits dist/ and .d.ts declarations
bun run test               # Vitest run (includes real loopback OAuth tests)
bun run test:coverage      # L1: >= 95% statements/branches/functions/lines
bun run release --dry-run  # Preview release bump & changelog only
```

`bun run build` emits `dist/`; typechecking alone does not produce a publishable package. Do not run `bun run release` without explicit release authorization.

## Invariants

- Publish compiled `dist/` with `.d.ts` declarations (`package.json` `files: ["dist"]`); never publish `src/`. Keep `dist/` gitignored and validate the built package before publication.
- Config files written by `ConfigManager` must preserve mode `0600`; consumers own their config directories and credentials.
- OAuth callbacks must bind to loopback only, verify state nonces, escape reflected HTML, and preserve the localhost-to-IPv4 fallback.
- Preserve dependency injection for browser opening, token persistence, and login URL configuration; tests must never log into a real service.
- Version lives in `package.json` (displayed with a `v` prefix); never publish a version absent from the remote branch, and verify with `npm view @nocoo/base-cli version` and GitHub Releases.
- Authorized maintainers build, verify lint/coverage, and review version/changelog before publishing.
- `scripts/release.ts` (`patch` default, `minor`, `major`, `<ver>`, `--dry-run`) handles version bump, changelog, commit/tag/push, npm publish, and GitHub release.
- Release defect: `scripts/release.ts` omits `build` and uses `--no-verify`; fix both before publishing and never bypass hooks. Raise weaker checks rather than lowering this contract.

## Quality & Local Gates

Follows `system1-quality`; CI slot definitions and pinned `base-ci` revision live in [.github/workflows/quality-slots.yml](.github/workflows/quality-slots.yml) via [.github/workflows/ci.yml](.github/workflows/ci.yml) (`quality / Z1: Gate` required on `main`). Status vocabulary: `enforced`, `planned`, `manual`, `N/A`.

- **Coverage boundary**: `L1 Unit` enforces >=95% statements/branches/functions/lines, no `.skip`/`.only`; `vitest.config.ts` excludes `scripts/**`, `src/index.ts` (barrel), and `src/update-command.ts` (gap).
- **Exceptions / gaps**: `L2 Runtime` is `N/A` (not a Worker); `L3 Integration` is `N/A` (loopback HTTP tested in L1); `L4 E2E` is `N/A` (no browser UI); `P1 Package` is `planned` (`npm pack`/`publint` not wired).
- **Local hooks**: installed via `bun install` (`prepare: husky`, `.husky/`); hooks are check-only. Verify hook installation in fresh checkouts.
- **pre-commit**: runs on a staged Git index snapshot (`gitleaks protect --staged`, `typecheck`, `lint`, `test:coverage`) with temp-dir cleanup; keep warm path under 30s.
- **pre-push**: runs working-directory `gitleaks` and `osv-scanner` (`osv-scanner.toml`, `bun.lock`); known gap: fail closed on missing scanners and inspect stdin push refs (<3min).

## Retrospective

Record incident narratives in [Retrospective.md](Retrospective.md); promote recurring project constraints to `Invariants` above and enforceable safeguards to tests/hooks.
