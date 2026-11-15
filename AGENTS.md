# base-cli

Shared CLI infrastructure (`@nocoo/base-cli`): config, OAuth loopback login, update checks, version, browser opening and logging.
Profile: cli-library.
Direction: [README.md](README.md). Frameworks must not rewrite this file.

## Scope and instruction sources

- This file is the only project handbook; nested files do not compete with it. Do not create a `CLAUDE.md` alias or copy.
- Source of truth: `package.json`, `tsconfig.json`, `biome.json`, `vitest.config.ts`; `.husky/` and `.github/workflows/ci.yml` enforce the contract below. Raise weaker enforcement instead of lowering this contract.
- Version lives in `package.json` and displays with a `v` prefix. Release tooling: `scripts/release.ts`, `CHANGELOG.md`. Accidents: [Retrospective.md](Retrospective.md).
- Global machine rules live in the machine `AGENTS.md` and `rules/`; this file adds only project specifics.

## Project invariants

- Publish the built `dist/` package with declarations; never publish `src/`. Build output remains gitignored.
- Config files written by this library use mode `0600`; consumers own their config directories and credentials.
- OAuth callbacks bind to loopback only, verify state nonces and escape reflected HTML. Preserve the localhost-to-IPv4 fallback.
- Keep dependency injection for browser opening, token persistence and login URL configuration; tests must not log into a real service.
- The release script does not build and currently bypasses Git hooks. That is a release-path defect; never use it to bypass normal commit or push checks.

## Setup and commands

Strict TypeScript 7 (the compiler excludes test files) on a Bun/Node runtime; Biome static checks with `--error-on-warnings`; Vitest with V8 coverage including real loopback OAuth requests. Layout: `src/` modules with colocated tests plus `scripts/release.ts`. CI pins Bun 1.3.14. Gitleaks and OSV Scanner are required for security checks; no production credentials are needed for tests.

Run from the repository root after a frozen Bun install:

```bash
bun install --frozen-lockfile
bun run typecheck
bun run lint
bun run build
bun run test
bun run test:coverage
bun run release --dry-run
```

`build` emits `dist/`; typechecking alone does not prepare a publishable package. Do not run the release command without a release request.

## Testing and quality contract

Quality follows `system1-quality` (v1.3.0) with 12 canonical slots (`impl`, `todo`, `na`) and `nocoo/base-ci` v2026.15 (`777bbe4f9bacab8a91d8c01974480a6ee23d28e0`) in native mode (`quality-slots.yml`).

| Slot | Required proof | Status | Current enforcement / gap |
| --- | --- | --- | --- |
| S1: Types | Strict TypeScript (`bun run typecheck`) | enforced (`impl`) | Husky pre-commit index snapshot & CI `s1-types` |
| S2: Lint | Biome check-only (`bun run lint`) | enforced (`impl`) | Husky pre-commit index snapshot & CI `s2-lint` |
| S3: Workflow | `actionlint` + 40-char SHA pinning | enforced (`impl`) | CI `s3-workflow` |
| G1: Secrets | Gitleaks scan (`.gitleaks.toml`) | enforced (`impl`) | Staged `gitleaks` in pre-commit; pre-push & CI `g1-secrets` |
| G2: Dependencies | `osv-scanner` (`osv-scanner.toml`, `bun.lock`) | enforced (`impl`) | Pre-push & CI `g2-dependencies` |
| B1: Build | Fresh emitted library and declarations (`bun run build`) | enforced (`impl`) | CI `b1-build` |
| L1: Unit | Vitest statements/branches/functions/lines each ≥95%, no `.skip`/`.only` | enforced (`impl`) | Husky pre-commit index snapshot (`test:coverage`) & CI `l1-unit` (`src/update-command.ts`, `src/index.ts`, `scripts/**` excluded by design) |
| L2: Runtime | Cloudflare Worker / platform runtime bindings | N/A (`na`) | Non-Cloudflare Worker project without workerd/miniflare tests |
| L3: Integration | Standalone HTTP service integration | N/A (`na`) | OAuth loopback HTTP is covered in L1 Vitest |
| L4: E2E | Browser / Playwright E2E | N/A (`na`) | CLI foundation library without browser UI |
| P1: Package | `npm pack` / `publint` validation | planned (`todo`) | `@nocoo/base-cli` CI package gate not yet wired |
| Z1: Gate | Aggregate all `impl` slots & SHA consistency | enforced (`impl`) | CI `z1-gate` (`quality / Z1: Gate`) |

| Hook | Current behavior | Required follow-up |
| --- | --- | --- |
| pre-commit | Index snapshot: gitleaks, typecheck, lint, `test:coverage` (four metrics ≥95%); temp dir cleanup | Warm-path timing verified locally; keep under 30s |
| pre-push | Working-directory Gitleaks and OSV on `bun.lock` | Fail on missing scanners; check stdin push refs, <3min |

`bun install` runs the existing Husky prepare step; verify hooks are installed in a new checkout. Hooks are check-only. Never use `--no-verify` on commits or branch pushes.
CI runs `.github/workflows/ci.yml` -> `./.github/workflows/quality-slots.yml` (`v2026.15` composite actions), gated on `quality / Z1: Gate`.

## Operations / release

Authorized npm/GitHub maintainers build, verify coverage/lint, review version/changelog and publish the intended `dist/` package. The existing entry is `bun run release` (patch default, or `minor`, `major`, explicit version, `--dry-run`), but its hook-bypass and missing-build behavior must be corrected before using it for publication.
It manages version/changelog, Git commit/tag/push, npm publication and GitHub release. Never publish a version that is absent from the remote branch. Check the published result with `npm view @nocoo/base-cli version` and its GitHub release.

## Retrospective

Narratives remain in [Retrospective.md](Retrospective.md); keep only concise recurring project lessons here. Move cross-project lessons to global rules/nmem and deterministic requirements to hooks/tests.
- Preserve private config permissions and validate the built package before publication.
