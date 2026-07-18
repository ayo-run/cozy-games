# Cozy Games

Shared, reusable packages that power small browser games — extracted from
[mnswpr](https://github.com/ayo-run/mnswpr) and published under the
`@cozy-games/*` scope on npm.

# Roadmap

- Stabilize the public APIs: game-agnostic modules for core, move-log, replay,
  leaderboard, and rating.
- Add a second game to validate those APIs through the adapter alone.
- Extract the proven modules into standalone, versioned packages.
- Freeze the adapter contract so third-party games can build against it.

## Packages

| Package              | Develop        | Publish                    |
| -------------------- | -------------- | -------------------------- |
| `mnswpr` (game core) | Built          | @cozy-games/mnswpr         |
| leaderboard          | Built          | @cozy-games/leaderboard    |
| move-log             | Built          | @cozy-games/move-log       |
| replay engine        | Built          | not yet published          |
| utils                | Built          | @cozy-games/utils          |
| rating math          | In development | not yet published          |
| `sudoku` (game core) | Planned        | not yet published          |

> The mnswpr game app itself lives in its own repo:
> [ayo-run/mnswpr](https://github.com/ayo-run/mnswpr).

## Contributing

Setup, testing, code style, and local infra all live in
**[CONTRIBUTING.md](CONTRIBUTING.md)**. In short — this is a
[pnpm](https://pnpm.io) workspace:

```bash
pnpm i                     # install
pnpm test                  # run the test suite
pnpm build                 # build all packages
```

See each package's README for library usage.

## License

MIT © Ayo Ayco
