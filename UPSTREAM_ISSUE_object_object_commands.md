# Upstream Issue: `[object Object]` In Server Commands

## Problem
Mindcraft sometimes sends malformed Minecraft commands containing `[object Object]` where a player name/string should be.

## Confirmed Evidence
From [`compose.full.fresh.wrong-setup.log`](compose.full.fresh.wrong-setup.log):

- `/tp bot_1 [object Object]` at [line 517](compose.full.fresh.wrong-setup.log:517)
- `/clear [object Object]` at [line 770](compose.full.fresh.wrong-setup.log:770)
- `/clear [object Object]` at [line 775](compose.full.fresh.wrong-setup.log:775)

This indicates non-string values are being interpolated into chat commands.

## Likely Upstream Code Areas
In upstream `mindcraft` task code:

- Success path clears every available agent with `/clear ${agent}`:
  - [tasks.js:372](../mindcraft/src/agent/tasks/tasks.js:372)
- Teleport logic uses first available agent in string interpolation:
  - [tasks.js:536](../mindcraft/src/agent/tasks/tasks.js:536)

If `available_agents` contains objects in some paths, JS coercion produces `[object Object]`.

## Reproduction Notes
1. Run task mode and capture full compose logs.
2. Grep for malformed command patterns:
   - `/tp .*\\[object Object\\]`
   - `/clear \\[object Object\\]`
3. Correlate with `available_agents` update points.

## Suggested Fix (Minimal)
Before command construction, normalize agent identifiers:

1. Accept strings directly.
2. If object, extract `.username` (or `.name`) if string.
3. Drop invalid entries and log warning.
4. Use normalized array for `/tp` and `/clear`.

## Suggested Diagnostic Logging
Add temporary logs before command emission:

- Raw `available_agents` payload type/value
- Normalized agent names list
- Final command string to be sent

This should make root cause visible in one run.
