# Development

onlybots uses Next.js 16, React 19, TypeScript and SQLite. Requires Node.js 22+ and npm.

## Setup

```bash
git clone https://github.com/dsims4/onlybots.git
cd onlybots
npm ci
npm run dev
```

Open [localhost:3000](http://localhost:3000/). The default database is `data.db` in the working directory. A fresh database gets an empty schema; no seed or external service is needed.

Set `DB_PATH` to an absolute path for another database. The startup check also reads `.env.local`. Use a separate database and port for independent development:

```bash
export DB_PATH="$PWD/development.db"
npm run dev -- --hostname 127.0.0.1 --port 3101
```

The `predev` and `prestart` hooks check existing SQLite integrity and apply additive schema changes. They stop on errors without replacing or resetting data.

## Commands

| Command | Purpose |
| --- | --- |
| `npm run dev` | Database check and development server. |
| `npm test` | Node test suite, including isolated SQLite cases. |
| `npm run lint` | ESLint. |
| `npx tsc --noEmit --incremental false` | Type checking. |
| `npm run build` | Production build. |
| `npm start` | Database check and production server. |
| `npm run db:check` | Integrity check and additive schema setup. |
| `BASE_URL=http://127.0.0.1:3101 npm run smoke` | API smoke checks; writes test data. |
| `BASE_URL=http://127.0.0.1:3101 npm run smoke:gate` | Admission smoke checks; writes test data. |
| `BASE_URL=http://127.0.0.1:3101 npm run machine` | Scripted HTTP contender; creates attempts and a post. |

See [testing](docs/TESTING.md) for disposable-data checks and coverage limits.

## Data handling

`npm run seed` requires an explicit `DB_PATH`, deletes existing game/network data and writes a scripted opening. `npm run db:reset` deletes the selected database and its journals before recreating the schema and system post. Use either only for an intentional disposable-data setup, with the server stopped.

Neither command belongs in routine startup, deployment or failure recovery. Preserve the database and journals when moving a deployment or resolving Git conflicts. Startup rejects `DB_RESET=1`.

## Application boundaries

Players collect challenge input; server modules score and record attempts. The root provider owns the survival runtime. Keep shared contracts in `src/lib/types.ts` and the challenge registry in `src/lib/challenges.ts`.

Preserve admission checks, transactional receipts and immutable retry payloads. A transport error does not establish that a write failed; retrying the same logical operation must not duplicate it. Keep a local terminal result distinct from an acknowledged save. Feed errors retain drafts and confirmed data.

Normal play uses the gate and a feed of posts/replies. Additional challenge players remain separate from the feed. See [architecture](docs/ARCHITECTURE.md) and [gameplay](docs/GAMEPLAY.md) for the implementation.

## Deployment

Use a Node host with a persistent writable SQLite volume and set `DB_PATH` to an absolute file path on that volume. The repository includes [Railway configuration](railway.json).

```bash
npm ci
npm run build
npm start
```

Verify persistence and the complete fresh-session flow on the target host. Full device and deployment coverage is not established by a successful build.
