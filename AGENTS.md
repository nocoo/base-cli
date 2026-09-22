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

6DQ keeps its name with unified L1, L2/L3, G2 and D1; the owner merged former G1 into L1 on 2026-09-21.
Statuses: `enforced` (configured execution with evidence), `planned` (required but incomplete), `manual`, `N/A`.

| Dimension | Required proof | Status | Current enforcement / gap |
| --- | --- | --- | --- |
| L1 pre-commit quality | UT with statements, branches, functions and lines each ≥95%, no `.skip`/`.only`; strict types and check-only lint/format with zero errors/warnings | planned | CI runs `test:coverage` with four 95% thresholds; installed pre-commit and CI run typecheck/lint (strict TS excludes test files). `src/update-command.ts`, `src/index.ts` and `scripts/**` are configured coverage exclusions by design. Pre-commit tests the working tree without coverage thresholds, so index-snapshot, <30s timing and rejection are unverified |
| L2 callback integration | Real HTTP for every callback endpoint/method and failure path | planned | `src/login.test.ts` sends real loopback HTTP in the normal Vitest suite; no complete method inventory or dedicated pre-push L2 gate exists |
| L3 user workflow | Real consumer CLI login/config/update journey | planned | No shipped standalone CLI or process-level consumer acceptance suite; absence of a browser runner does not make this unnecessary |
| G2 security | Dependency and secret scans; missing tools fail | enforced | CI uses pinned `base-ci/quality.yml` with both scanners; local pre-push still skips missing Gitleaks and scans the working directory |
| D1 isolation | Per-run temporary state, loopback servers and guarded cleanup | planned | Config/version tests use temporary directories and login uses ephemeral ports; timestamps are not exclusive allocation and cleanup lacks a common guard |
| Build | Fresh emitted library and declarations | enforced | CI `prepare-command: bun run build` |
| Docs / release | README API changes, version and changelog reviewed | manual | Maintainer review; release hook-bypass defect must be fixed before relying on automation |

| Hook | Current behavior | Required follow-up |
| --- | --- | --- |
| pre-commit | Working-tree typecheck, lint, tests without coverage; staged Gitleaks | Unified L1 on an index snapshot, <30s |
| pre-push | Working-directory Gitleaks and OSV on `bun.lock` | Fail on missing scanners; check stdin push refs and run L2, <3min |

`bun install` runs the existing Husky prepare step; verify hooks are installed in a new checkout. Hooks are check-only. Never use `--no-verify` on commits or branch pushes.
CI pins `nocoo/base-ci/.github/workflows/quality.yml@ad43150de3a2be2fa464b5cd2f921dc4fa9f8f0f`; do not substitute a moving tag.

## Operations / release

Authorized npm/GitHub maintainers build, verify coverage/lint, review version/changelog and publish the intended `dist/` package. The existing entry is `bun run release` (patch default, or `minor`, `major`, explicit version, `--dry-run`), but its hook-bypass and missing-build behavior must be corrected before using it for publication.
It manages version/changelog, Git commit/tag/push, npm publication and GitHub release. Never publish a version that is absent from the remote branch. Check the published result with `npm view @nocoo/base-cli version` and its GitHub release.

## Retrospective

Narratives remain in [Retrospective.md](Retrospective.md); keep only concise recurring project lessons here. Move cross-project lessons to global rules/nmem and deterministic requirements to hooks/tests.
- Preserve private config permissions and validate the built package before publication.
