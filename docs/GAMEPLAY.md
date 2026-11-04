# Gameplay

onlybots asks players to behave like machines for as long as possible. Verification and survival use separate clocks and results.

## Gate

Enter a maker, model and version, such as `acme atlas 1.0`. The designation is normalized to `acme-atlas-1.0`. The inline reverse CAPTCHA asks for a category and its look-alike; the server evaluates the issued challenge and submitted evidence before granting admission.

Current CAPTCHA rules:

- Select matching tiles, including the look-alike category. One total missing/extra tile is allowed if at least one correct tile is selected.
- One deselection correction is allowed.
- Two bad pointer strokes are allowed; the third fails.
- Keep every pause under five seconds.
- Finish the round within 30 seconds.

A confirmed pass opens the feed. A measured failure shows its reason. An uncertain request retains its submission for retry rather than inventing a verdict.

## Survival

Entering the admitted feed automatically starts a server-backed run and a three-second countdown. The HUD shows time, stage, input measurements and the current objective.

Post and like objectives alternate. A post must satisfy the unit’s current rotating content rule. Likes must target eligible posts; if none remain, the game replaces the like objective with a new post objective and full budget. Replies have separate content/typing checks and are not survival objectives.

Post composition accepts pasted/edited text and is exempt from typing-rhythm scoring. Other supported editable fields retain typing checks. Pointer scoring applies to the pointer input mode; coarse-pointer devices use the touch/keyboard mode.

## Difficulty

The current rules version is `survival-v4`. These are gameplay settings, not validated human-classification thresholds.

| Active time | Stage | Idle limit | Pointer ratio | Typing CV | Minimum WPM | Scroll CV | Post/like budget | Bad windows to fail per detector |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0–<20s | Boot | 5s | 0.08 | 0.35 | 45 | 0.60 | 15s | 2 |
| 20–<40s | Observe | 4s | 0.06 | 0.25 | 55 | 0.45 | 12s | 1 |
| 40–<50s | Inspect | 3.5s | 0.05 | 0.22 | 60 | 0.40 | 10s | 1 |
| 50–<60s | Audit | 3s | 0.045 | 0.20 | 65 | 0.35 | 9s | 1 |
| 60s onward | Purge | 2.5s | 0.04 | 0.18 | 70 | 0.30 | 8s | 1 |

CV is standard deviation divided by mean. Insufficient evidence is unscored. Good sensor windows leave accumulated bad-window counts in place. Objectives retain their issued budget; sensor windows retain their captured stage.

Exact values and sample bounds live in [config.ts](../src/lib/survival/config.ts). The gate’s selection and behavioral rules live in [image-captcha.ts](../src/lib/image-captcha.ts) and [captcha-verdict.ts](../src/lib/captcha-verdict.ts).

## Results and recovery

The first terminal failure freezes gameplay and its evidence. Same-time failure precedence is verification, objective deadline, idle, pointer, typing, then scroll. Manual End game produces an unranked interruption.

The browser saves a terminal snapshot through the run API. Saving, saved and failed-save states remain distinct. Retry retains the same request ID and snapshot. Start again waits for saving, ends browser admission and returns to the gate without deleting earlier posts or results.

Pending objective delivery pauses the engine. Confirmation advances once; definite rejection retains the objective/deadline and releases the request pause; uncertain delivery keeps the immutable request for explicit retry.

The current runtime does not automatically pause for hidden tabs, lost focus or leaderboard visits. Reload discards the checkpoint, and a full feed load returns to the gate. Cross-refresh recovery of unsaved results is not provided.

## Rankings and detections

The leaderboard shows longest failed runs, fastest successful image verifications and recent detections. Manual interruptions are unranked.

The public longest-run query groups across rules versions/input modes and independently aggregates duration and objective count. The separate `/api/scores?inputMode=pointer` API selects a whole best run within a version/mode. These queries are not interchangeable.

Rejected feed actions also accumulate designation detections; three detections block further writes. That mechanism is separate from survival detector counts.

Browser measurements are forgeable and designations are not authenticated accounts. Device fairness, lifecycle recovery and ranking comparisons need broader verification; see [TESTING.md](TESTING.md).
