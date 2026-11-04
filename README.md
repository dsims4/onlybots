# onlybots

**A social network with one rule: no humans.**

Built for Showerhacks, onlybots turns a familiar social feed into a human-versus-machine survival game. Players invent a machine designation, pass a reverse CAPTCHA, and try to keep posting and liking while the network demands increasingly machine-like behavior.

Developed collaboratively for Showerhacks, this project combines interactive game design, server-side scoring and a scripted machine contender. Based on the [original team project](https://github.com/seed0x/bot-only).

## The experience

1. **Identify yourself.** Enter a maker, model and version, such as `acme atlas 1.0`.
2. **Prove you’re not human.** Complete an image challenge that checks selections, corrections, pauses and pointer movement on the server.
3. **Survive the feed.** Admission opens the feed and automatically starts a survival run. Complete post and like objectives before their deadlines while movement, inactivity and other supported input are monitored.
4. **Face the verdict.** A failed run shows its measured cause and saves its result. Rankings show longest survival runs and fastest successful verifications. Manual End game records an unranked interruption.

The feed contains posts and replies, rather than embedded challenge cards. Posts follow rotating content rules; post composition is exempt from typing-rhythm checks.

## Engineering highlights

- **Server-scored admission:** the browser submits challenge evidence; a recorded image-CAPTCHA pass establishes an HttpOnly admission cookie checked by the feed and its write endpoints.
- **A game engine outside React:** pure state transitions and detector functions handle time, objectives and failures. A root provider connects them to browser input and UI.
- **Retryable mutations:** SQLite transactions store domain writes and their receipts together. Retrying the same request ID and payload replays the acknowledgement; conflicting reuse is rejected.
- **An independent machine contender:** a separate HTTP client solves issued challenges and posts through the application API, without direct database access.

**Stack:** Next.js 16 App Router · React 19 · TypeScript · Tailwind CSS 4 · SQLite through Node’s built-in `node:sqlite` · Node’s test runner.

See the [architecture walkthrough](docs/ARCHITECTURE.md) for implementation details and source entry points.

## Run locally

Requires Node.js 22+ and npm. Use the committed lockfile.

```bash
git clone https://github.com/dsims4/onlybots.git
cd onlybots
npm ci
npm run dev
```

Open [localhost:3000](http://localhost:3000/). No external database service or API key is required. Startup creates an empty SQLite schema at `data.db`; you can create content through normal play. Set `DB_PATH` to choose another file.

Startup checks existing data and applies additive schema changes. It stops on integrity or migration errors. Seeding and resetting are separate, destructive commands; neither is required to try the app. See [local development and deployment](CONTRIBUTING.md).

## Checks

```bash
npm test
npm run lint
npx tsc --noEmit --incremental false
npm run build
```

The suite includes scoring, state-machine, browser-adapter and SQLite integration tests. Some cases are explicitly skipped; a passing command does not establish full gameplay or device coverage. API smoke scripts write data and require a disposable server. See [testing and known coverage gaps](docs/TESTING.md).

## Hackathon scope

Behavioral measurements are game mechanics, not validated human classification or tamper-proof anti-cheat. Survival telemetry comes from the browser. Designations are not authenticated accounts, and input fairness across phones and trackpads still needs broader playtesting.

Current rules, lifecycle behavior and ranking limits are documented in [gameplay](docs/GAMEPLAY.md).

## Documentation

- [Architecture](docs/ARCHITECTURE.md) — system design, data flow and code map.
- [Contributing](CONTRIBUTING.md) — setup, commands and data handling.
- [Gameplay](docs/GAMEPLAY.md) — rules, difficulty and result recovery.
- [Testing](docs/TESTING.md) — verification commands and coverage limits.
