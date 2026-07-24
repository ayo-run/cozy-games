# AGENTS.md

Guidance for all AI coding AGENTS working in this repository.

## What this is

**Cozy Games** — a pnpm workspace of small, framework-free, publishable `@cozy-games/*`
packages that power browser games. **This repo is libraries only.** The playable
mnswpr *website/app* (Firebase, Netlify, `apps/`) was extracted to its own repo,
[ayo-run/mnswpr](https://github.com/ayo-run/mnswpr) — so there is **no `apps/`
directory, no Firebase/Netlify infra, and no dev/deploy scripts here**, despite
what older `CONTRIBUTING.md` sections still say (they carry stale app-era content;
trust the code and this file over them).

Everything is vanilla JS — no framework, no TypeScript source: `// @ts-check` +
JSDoc only. The game core and utils have **zero runtime dependencies**.

## Commands

All commands run from the repo root (pnpm is required — npm/yarn will not work):

```bash
pnpm i               # install
pnpm test            # run the whole Vitest suite once (jsdom)
pnpm test:watch      # Vitest in watch mode
pnpm lint            # eslint . (lints **/*.js AND **/*.css); runs on pre-commit
pnpm lint:fix        # eslint --fix
pnpm build           # build:types, then build every package (pnpm -r)
pnpm build:types     # regenerate committed .d.ts from JSDoc (scripts/build-types.mjs)
pnpm build:lib       # build just the publishable mnswpr engine -> packages/mnswpr/dist
```

Run a single test file / test by name:

```bash
pnpm test packages/mnswpr/test/<file>.test.js     # one file
pnpm test -t "chording reveals neighbors"         # by test name (substring)
```

There is **no app to run** in this repo. For anything visual/input-timing you must
verify by playing, do it in the separate mnswpr app repo. Node version is pinned
by `.nvmrc` (`lts/*`).

## Packages (workspace = `packages/*` only)

- **`@cozy-games/mnswpr`** (`packages/mnswpr/`) — the standalone Minesweeper engine,
  published to npm. Split into a **headless core** and a **DOM client** (see
  Architecture). `mnswpr.js` is the browser default entry (`.`); the core is the
  `./core` sub-path of the *same* package (not a second published package).
- **`@cozy-games/leaderboard`** (`packages/leaderboard/`) — a backend-agnostic,
  time-windowed leaderboard shipped as a **web component** (`web-component-base`),
  with storage injected via **adapters** in `adapters/` (`firebase`,
  `firebase-admin`, `supabase`). `firebase` is a peer dependency.
- **`@cozy-games/move-log`** (`packages/move-log/`) — a game-agnostic, schema-versioned
  container for an ordered stream of move events (`{ seq, clientTs, type, payload }`).
  It never inspects a `payload`. This is the interchange format between the engine
  and replay.
- **`@cozy-games/replay`** (`packages/replay/`) — a game-agnostic replay engine.
  `PlaybackClock` re-drives a move-log envelope over time (`play`/`pause`/`seek`),
  interpreting nothing itself; a game **adapter** supplies `progress(events) → %`.
  Currently `private` (unpublished).
- **`@cozy-games/utils`** (`packages/utils/`) — zero-dependency shared services,
  re-exported from `index.js`: `StorageService`, `TimerService` (`pretty()` time
  formatting), `LoggerService`, `LoadingService`, and date-bucket helpers.

Packages import each other by name via `workspace:*` (e.g. `@cozy-games/utils`), never
by relative path across package boundaries.

## Architecture

**Read the design docs and ADRs before large changes.** The intended structure and
the reasoning are recorded, not just implied:
- `packages/mnswpr/docs/headless-core-and-client-design.md` — the core/client split.
- `docs/decisions/` — ADRs: `0001-package-boundary`, `0002-game-adapter-pattern`,
  `0003-stored-boards-not-seeds`.

**mnswpr = headless core + thin DOM client.** The engine is deliberately split so the
same logic runs in a browser (offline play) or on an authoritative host (server-side
timing / replay verification):

- **`core/`** — headless, isomorphic, **zero DOM, zero wall-clock**. Internally
  layered so the generic bottom can later be lifted into `@cozy-games/grid` +
  `@cozy-games/game-session` once a second game exists to validate it. `core/index.js`
  is the public barrel:
  - `core/grid/` (Layer 0) — generic `Grid`, neighbor strategies (`eightWay`,
    `orthogonal`), serialize.
  - `core/session/` (Layer 1) — `GameSession` (lifecycle, injected clock, move log),
    seedable PRNG (`mulberry32`), `replay()` validation.
  - `core/minesweeper/` (Layer 2) — `MinesweeperRules`, deterministic board gen with
    first-click safety (`board.js`), flood-fill + chording (`reveal.js`).
- **`client/`** — DOM internals that consume the core: `renderer.js` (the **only**
  place `document` is touched — events → DOM) and `transport.js` (`LocalTransport`).
- **`mnswpr.js`** — the DOM client entry. It still holds the **intricate input state
  machine** inline (mouse left/right/middle + left+right "chording", touch long-press
  to flag), debounced by `isBusy` (`MOBILE_BUSY_DELAY`/`PC_BUSY_DELAY`). This is
  deliberately not yet extracted and is under-tested — **tread carefully; small
  changes easily break chording or mobile flagging**. `levels.js` holds the four
  difficulty presets, shared by client and core.

**The engine is decoupled from any app via two hooks.** `Minesweeper(appId, version,
hooks, options)` injects app behavior through `hooks.levelChanged(setting)` and
`hooks.gameDone(game)`. When adding engine features an app needs to react to, add a
new hook rather than reaching outward — that separation is what keeps the package
publishable on its own. `options.seed` pins a deterministic board (tests/replay).

**Test mode:** set `TEST_MODE = true` at the top of `packages/mnswpr/mnswpr.js` to
render mine positions as visual hints and enable debug logging.

**Game/replay data flow:** engine move events → `@cozy-games/move-log` envelope →
`@cozy-games/replay` `PlaybackClock`. The move-log and replay engine are strictly
game-agnostic; anything game-specific is confined to an **adapter** (see ADR-002 and
`packages/mnswpr/adapters/` — `replay-common`, `replay-progress`, `replay-state`).

## Conventions

- **Style is ESLint Stylistic, not Prettier:** 2-space indent, single quotes, **no
  semicolons**, no trailing commas, spaces inside `{ braces }` but not `[brackets]`.
  Both `**/*.js` and `**/*.css` are linted (CSS via `@eslint/css`). Run `pnpm lint:fix`
  before committing.
- **Match the file you edit:** the engine uses plain functions and `var`/`let`
  closures; `packages/utils/` uses ES classes.
- **Types are generated, not authored.** Source is JS + JSDoc (`// @ts-check`). `tsc`
  emits `.d.ts` co-located next to each source file, and those declarations are
  **committed**. After changing JSDoc on a published file, run `pnpm build:types` and
  commit the regenerated `.d.ts`. A new published source file only ships types if it's
  added to `tsconfig.types.json`'s `include` list.

## Content policy (enforced — will block commits/CI)

Commit messages, branch names, PR text, and contributed lines are scanned by
`scripts/check-content.mjs` (git hooks + the `Checks` workflow) against
`.repo-policy.json`:
- Write commit messages in plain project voice (conventional prefixes: `feat:`,
  `fix:`, `chore:`, …). **No AI-tool attribution trailers/footers**, no
  `Co-Authored-By:` for a non-human contributor (human co-authors are fine), no
  session links.
- The scanner also matches a maintainer-managed **reserved-terms list**; findings
  report a location + masked preview, never the term. If flagged, reword — **do not
  edit `.repo-policy.json`**, or ask a maintainer. For legitimate prose *about* these
  topics, a `content-policy: allow-next-line` comment above the line is the escape
  hatch.

## Git hooks & release (maintainer workflow)

- **Husky:** `pre-commit` runs `pnpm lint` + secret scan (`secretlint`) + the content
  check; `commit-msg` runs the content check over the message; **`post-commit`
  auto-pushes to extra remotes (`git push gh`, `git push sh`)** — if those remotes
  aren't configured locally, post-commit failures are environmental, not a code
  problem.
- **Release** (`pnpm release`, maintainers only) builds the lib, runs `bumpp` in
  `packages/mnswpr/`, then publishes. Only run when explicitly releasing.
