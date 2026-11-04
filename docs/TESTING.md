# Testing and verification

The repository includes pure scoring/state tests, browser-adapter harnesses and isolated SQLite integration checks. Browser and device coverage remains separate from these automated checks.

## Local checks

With the committed dependencies installed:

```bash
npm test
npm run lint
npx tsc --noEmit --incremental false
npm run build
```

`npm test` uses Node’s test runner and the repository’s SQLite/TypeScript flags. Tests cover challenge scoring, designation parsing, detector math, engine transitions, browser adapters, controlled component/provider harnesses, database startup, admission, mutation receipts and run persistence.

Some regression cases are explicitly skipped, including engine timing, detector thresholds, provider lifecycle/objectives and SQLite writes/rankings. Check the runner’s pass/fail/skip totals and review relevant `test.skip` cases before claiming coverage. Controlled hook/event harnesses are not mounted-browser or real-device tests. Do not disable rules or remove failing cases to claim a green baseline.

## API checks on disposable data

Smoke and machine scripts create real users, attempts, posts and other events. Never target a shared demo or production database for routine checks. The SQLite integration tests choose temporary paths; HTTP scripts write to whichever server `BASE_URL` selects.

Use a separate checkout with its own dependencies and build output. In terminal A, select a new temporary directory and free port:

```bash
export DB_PATH="$(mktemp -d)/onlybots-qa.db"
npm run dev -- --hostname 127.0.0.1 --port 3101
```

Startup creates the empty schema. Seeding is not required. In terminal B:

```bash
curl -fsS http://127.0.0.1:3101/api/health
BASE_URL=http://127.0.0.1:3101 npm run smoke:gate
BASE_URL=http://127.0.0.1:3101 npm run smoke
BASE_URL=http://127.0.0.1:3101 node scripts/leaderboard-smoke.mjs
BASE_URL=http://127.0.0.1:3101 npm run machine
```

Stop if a command fails and record the failure. The machine script is a separate contender, not a browser survival test. Keep its run ID/handle when following its immutable-operation retry instructions.

For production rehearsal, stop dev, build, then run `npm start -- --hostname 127.0.0.1 --port 3101` with the same disposable `DB_PATH`. Stop every test server you started after verification. Preserve pre-existing servers and data files; no reset/seed belongs in deployment verification.

## Browser and failure acceptance

| Area | Required observation |
| --- | --- |
| Admission | Designation alone cannot open the feed. Recorded pass admits; wrong/expired attempts do not. Direct anonymous feed/API access is rejected. |
| Mutation integrity | Lost response followed by identical retry records once. Changed payload with the same ID conflicts. Identity mismatch cannot replay another unit’s write. |
| Objectives | Post/like completion advances once after acknowledgement. Rejection retains objective/deadline. Uncertain delivery retains retry data. No eligible likes produces a new post objective. |
| Detectors | Demonstrate actual idle, pointer, typing, scroll and missed-objective failures where supported. Check current stage boundaries and precedence. |
| Content rules | Valid pasted/edited posts pass content checks; invalid content retains input and records the appropriate rejection. Reply rules remain separate. |
| Results | Failure freezes gameplay. Save error stays visible; identical retry persists once. Start again waits for saving before ending admission. |
| Navigation | Confirm current automatic feed start, no automatic tab/leaderboard pause and fresh-gate reload behavior. Verify that navigation does not reset difficulty during client-side transitions. |
| Rankings | Confirm stored results and displayed order. Review public longest-run aggregation versus version/mode-filtered scores. |
| Resource errors | Loading/empty/error/stale remain distinct. Outdated responses cannot erase newer state. No mock fallback or silent success. |

Use controlled transport failure, malformed responses, delays and dropped accepted-write responses on isolated data. Record both server data assertions and what the player sees.

## Responsive and input matrix

Inspect the gate, CAPTCHA, feed, game-over screen and leaderboard at 320×568, 375×812, 768×1024, 1024×768 and 1440×900. Include short landscape, software keyboard, 200% zoom, reduced motion, long designations and a 280-character unbroken post.

Check page width as well as visible wrapping, focus and touch targets. Cover loading, error, disabled and pending-save states. Exercise keyboard navigation, mouse, real trackpad inertia and touch; record unavailable devices as not run. A desktop screenshot does not establish responsive acceptance.

## Release rehearsal

Use a fresh session and record commit, URL, server mode and disposable database path. Complete designation → CAPTCHA failure/retry → admission → post/like → survival failure → saved result → restart. Rehearse the machine contender separately and verify hosting persistence without resetting live content.

Full browser, real-device and deployment coverage remains incomplete. Record untested cases explicitly; passing automated checks alone does not establish this coverage.
