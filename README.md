# Cozy Games

Shared, reusable packages that power small browser games — extracted from
[mnswpr](https://github.com/ayo-run/mnswpr) and published under the
`@cozy-games/*` scope on npm.

# Roadmap

- **Public APIs** — game-agnostic modules (core, move-log, replay, leaderboard, rating).
- **Second Game** — validate those APIs by adding a second game through the adapter alone.
- **Reusable Packages** — extract proven modules into standalone, versioned packages.
- **Adapters** — freeze the adapter contract for third-party games to build against.

## Packages

| Package              | Develop        | Publish                    |
| -------------------- | -------------- | -------------------------- |
| `mnswpr` (game core) | ✅ Built       | ✅ @cozy-games/mnswpr      |
| leaderboard          | ✅ Built       | ✅ @cozy-games/leaderboard |
| move-log             | ✅ Built       | ✅ @cozy-games/move-log    |
| replay engine        | ✅ Built       |                            |
| utils                | ✅ Built       | ✅ @cozy-games/utils       |
| rating math          | 🚧 Development |                            |
| `sudoku` (game core) | 🔮 Planned     |                            |

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
