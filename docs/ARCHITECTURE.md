# Architecture

onlybots runs as a Next.js application with React clients and a local SQLite database. The server issues and scores verification challenges, enforces admission and persists writes. The browser runs the survival simulation and submits its terminal evidence for server validation and storage.

```mermaid
flowchart LR
    Gate[Designation and reverse CAPTCHA] --> API[Next.js route handlers]
    Feed[Feed and composer] --> API
    Machine[Independent HTTP contender] --> API
    Provider[Root game provider] --> Engine[Pure engine and detectors]
    Provider --> API
    API --> Scoring[Challenge scoring and admission]
    API --> Receipts[Transactions and operation receipts]
    Scoring --> DB[(SQLite)]
    Receipts --> DB
```

## Admission and challenge scoring

The gate validates a maker/model/version designation and registers an unverified unit. `ChallengeTrial` requests a challenge instance, renders its player and submits bounded solution data for recording. The server checks the issued instance and evaluates the image selection and behavioral evidence.

A recorded image-CAPTCHA pass issues a random admission token in an HttpOnly cookie. SQLite stores its hash and associates it with the user and passing attempt. The feed’s server page checks admission before rendering; posts, replies and likes also check the caller’s identity. A supplied handle alone does not authorize a feed write.

The challenge registry also supports straight-line and hash-recall tests. These remain available through the shared challenge/API path and machine client; the normal feed does not render challenge cards.

## Survival runtime

One `GameProvider` in the root layout connects the pure engine to browser sensors, request lifecycles and result saving. The HUD automatically starts a run on entry to `/feed`, selects an input mode from the device’s coarse-pointer media query and shows a countdown. The provider stays mounted through client navigation.

The engine consumes timestamped events and returns immutable state. It keeps active time, stage, detector counts, objective deadlines and a terminal snapshot outside component markup. Pure detector functions evaluate bounded pointer, typing and scroll samples. Stage thresholds live in versioned configuration.

Post and like objectives receive completions from acknowledged server mutations. Pending objective delivery pauses the engine until confirmation, definite rejection or explicit retry. A failed run freezes locally before its save is confirmed. The result screen distinguishes saving, saved and error states; restarting saves first, then ends admission and returns to the gate.

The provider does not acquire automatic pauses for hidden tabs, lost focus or leaderboard visits, and it discards checkpoints on mount. See [gameplay](GAMEPLAY.md) for current lifecycle behavior.

## Durable writes and retries

`operation()` wraps a domain mutation and its acknowledgement in a SQLite transaction. Each logical write has a request ID and a fingerprint of its operation and payload:

- First use commits the mutation and serialized response together.
- Same ID and payload returns that response without repeating the write.
- Same ID with a different payload returns a conflict.

Run start, binding, finish and objective associations add validation around this mechanism. Feed mutations require admission before replay. Result-save retries retain the original request ID and snapshot, keeping local game over distinct from a confirmed stored result.

## Storage and deployment

SQLite stores users, issued challenges, attempts, hashed admission sessions, posts, replies, likes, activity, operation receipts, survival runs and objective events. `DB_PATH` selects one file. Startup checks integrity and applies additive schema changes; it does not seed or replace data after an error.

This keeps the hackathon deployment small: one Node process and a persistent writable volume. It does not establish horizontal scaling or a production account system. Supreme UI type is loaded from Fontshare; Archivo numeric type is bundled locally.

## Source map

| Path | Responsibility |
| --- | --- |
| [GateClient.tsx](../src/components/GateClient.tsx), [ChallengeTrial.tsx](../src/components/ChallengeTrial.tsx) | Designation, issue/play/record lifecycle and admission navigation. |
| [game.ts](../src/lib/game.ts), [image-captcha.ts](../src/lib/image-captcha.ts), [captcha-verdict.ts](../src/lib/captcha-verdict.ts) | Server challenge lifecycle and image verdict rules. |
| [gate.ts](../src/lib/gate.ts) | Admission tokens and server identity checks. |
| [survival/](../src/lib/survival/) | Pure engine, detectors, browser adapters, configuration and run/objective validation. |
| [game/](../src/components/game/) | Provider, HUD and terminal result UI. |
| [feed/](../src/components/feed/) | Composer, objectives, posts, replies and rankings. |
| [api/](../src/app/api/) | HTTP request handling and runtime validation. |
| [db.ts](../src/lib/db.ts), [operations.ts](../src/lib/operations.ts) | Schema and transactional mutation receipts. |
| [types.ts](../src/lib/types.ts), [challenges.ts](../src/lib/challenges.ts) | Shared contracts and challenge registry. |
| [machine.mjs](../scripts/machine.mjs) | Separate contender using the application’s HTTP endpoints. |
| [tests/](../tests/) | Scoring, engine, adapter, component-harness and SQLite checks. |

## Trust and verification limits

Server scoring prevents a client-provided pass flag from admitting a unit, but submitted browser measurements remain forgeable. Designations have no account authentication or recovery. The machine client demonstrates the protocol, rather than an independent AI model or robust anti-cheat test.

The public longest-run query combines maxima and groups across rules versions/input modes; the separate `/api/scores` query applies stricter version/mode filtering and selects a whole run. Do not describe the public table as equivalent to that API. Some regression tests are skipped, and full device/deployment acceptance remains open in [TESTING.md](TESTING.md).
